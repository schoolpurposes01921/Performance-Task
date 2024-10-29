/*
  Pathfinder script
*/

window.Pathfinder = {
  _enabled: true,
  _cache: null,

  _baseUrl: 'https://pathfinder.w3schools.com',
  _apiBaseUrl: 'https://api.kai.w3sapi.com/pathfinder',
  _mylApiBaseUrl: 'https://myl-api.w3schools.com',

  _current: {
    topicId: null,
    chapterId: null,
    lessonId: null,
    lessonFullName: null,
    lessonFinishedOnInit: null,
  }, // we get it from MyLearning script

  _userXpBumped: false,
  _userXp: null,
  _userGoalProgress: null,
  _goalMeta: null,
  _goalNavMeta: null,
  _nextLessonButtonClicked: false, // one time action
  // _nextLesson: null,
  _menuButtonClickProcessing: false,

  _uiTemplates: {},
  _uiMyPathRendered: false,
  _uiProgressUpdated: null,
  _menuModalRendered: false,
  _infoModalRendered: false,
};

Pathfinder._isDebugMode = function () {
  return Util.isDebugMode('Pathfinder');
};

Pathfinder._logDebug = function (message, data, logRawData) {
  return Util.logDebug('Pathfinder', message, data, logRawData);
};

Pathfinder._logError = function (message, data, logRawData) {
  return Util.logError('Pathfinder', message, data, logRawData);
};

Pathfinder.init = function () {
  Pathfinder._logDebug('init');

  if (Pathfinder._enabled === null) {
    Pathfinder._enabled = localStorage.getItem('Pathfinder.enabled') === 'true';
  }

  Pathfinder._logDebug('init -> enabled: ', Pathfinder._enabled);

  if (!Pathfinder._enabled) {
    return;
  }

  var _init = async (callback) => {
    var output = {
      error: {
        code: '1',
        description: 'Failed performing "Pathfinder._init"'
      },
      data: null,
    };

    var prepareOutput = (output) => {
      if (output.error.code === '1') {
        output.error = { code: '0' };
      }

      if (output.error.code !== '0') {
        Pathfinder._logDebug('init -> error -> output: ', output);
        Pathfinder.clearCache();
      }

      if (typeof callback !== 'undefined') {
        callback(output);
      }

      return output;
    };

    var cache = Pathfinder.getCache();

    var urlQueryPrs = new URLSearchParams(window.location.search);
    var urlActiveUserGoalId = urlQueryPrs.get('goalId');
    var pathfinderLearningPathMeta = null;

    if (urlActiveUserGoalId) {
      cache = Pathfinder.updateCache({
        activeUserGoalId: urlActiveUserGoalId,
      });
    }

    if (cache.activeUserGoalId) {
      var fetchPfLearningPathMetaRes = await Pathfinder.fetchPfLearningPathMeta({
        goalId: cache.activeUserGoalId,
        topicId: Pathfinder._current.topicId,
        chapterId: Pathfinder._current.chapterId,
      });

      Pathfinder._logDebug('init -> cache, fetchPfLearningPathMetaRes: ', {
        cache: cache,
        fetchPfLearningPathMetaRes: fetchPfLearningPathMetaRes,
      });

      if (fetchPfLearningPathMetaRes.error.code === '0') {
        // if (fetchPfLearningPathMetaRes.data.goal.status === 'active') {
        pathfinderLearningPathMeta = fetchPfLearningPathMetaRes.data;
        // } else {
        //   output.error = {
        //     code: 'GINA',
        //     description: 'Goal is not active',
        //     meta: {
        //       goalId: cache.activeUserGoalId,
        //       goal: fetchPfLearningPathMetaRes.data.goal,
        //     }
        //   };

        //   output.error.context = 'Pathfinder.fetchPfLearningPathMeta';
        //   return prepareOutput(output);
        // }
      } else {
        output.error = fetchPfLearningPathMetaRes.error;
        output.error.context = 'Pathfinder.fetchPfLearningPathMeta';
        return prepareOutput(output);
      }
    } else {
      output.error = {
        code: 'NO_ACTIVE_GOAL',
        description: 'No active goal',
        context: 'Pathfinder.fetchPfLearningPathMeta',
      };

      return prepareOutput(output);
    }

    Pathfinder._goalMeta = pathfinderLearningPathMeta.goal;
    Pathfinder._goalMeta.uuid = cache.activeUserGoalId;
    Pathfinder._userXp = pathfinderLearningPathMeta.userXp;
    Pathfinder._userGoalProgress = pathfinderLearningPathMeta.userGoalProgress;

    var promises = [];

    // cache my path template
    promises.push(Pathfinder.uiFetchTemplate('/lib/pathfinder/html/user-authenticated/my-path.tmpl.html'));

    // Check for cookie and possibly initiate modal rendering
    var hideMyPathInfoModal = Cookies.get('Pathfinder.hideMyPathInfoModal');
    if (hideMyPathInfoModal !== 'true') {
      promises.push(Pathfinder.uiRenderMyPathInfoModal());
    }

    await Promise.all(promises);

    Pathfinder.uiRenderMyPath();

    return prepareOutput(output);
  };

  Util.objFieldOnSetCallback({
    scopeRef: MyLearning,
    fieldName: 'topicId', // set by MyLearning only after it's fetched from the server
    callback: function (res) {
      if (res.error.code === '0') {
        Pathfinder._current = {
          topicId: MyLearning.topicId,
          chapterId: MyLearning.chapterId,
          lessonId: MyLearning.lessonId,
          lessonFullName: MyLearning.lessonFullName,
          lessonFinishedOnInit: MyLearning.lessonFinishedOnInit,
        };

        Pathfinder._userXpBumped = MyLearning.lessonFinishedOnInit === true;

        Pathfinder._logDebug('Pathfinder._current: ', Pathfinder._current);

        if (!Util.allObjectFieldsAreSet(Pathfinder._current)) {
          Pathfinder._logError('MyLearning.topicId -> missing required parameters -> Pathfinder._current: ', Pathfinder._current);
          return;
        }

        _init((res) => {
          if (res.error.code === 'FFUPGBID') {
            _init();
          }
        });
      } else {
        Pathfinder._logError('MyLearning.topicId -> res: ', res);
      }
    }
  });
};

Pathfinder.clearCache = function () {
  Pathfinder._cache = null;

  sessionStorage.removeItem('Pathfinder.cache');
};

Pathfinder.getCache = function () {
  if (Pathfinder._cache !== null) {
    return Pathfinder._cache;
  }

  var rawCache = sessionStorage.getItem('Pathfinder.cache');

  if (rawCache === null) { // not set
    Pathfinder._cache = {};
  } else {
    var parseJsonRes = Util.parseJson(rawCache);

    if (parseJsonRes.error.code !== '0') {
      Pathfinder._logError('getCache -> parseJsonRes: ', parseJsonRes);
    }

    Pathfinder._cache = parseJsonRes.data;
  }

  return Pathfinder._cache;
};

Pathfinder.updateCache = function (data) {
  var cacheRef = Pathfinder.getCache();

  try {
    for (var key in data) {
      if (data.hasOwnProperty(key)) {
        cacheRef[key] = data[key];
      }
    }

    cacheRef.updatedUtms = Util.getCurrentUtms();

    sessionStorage.setItem('Pathfinder.cache', JSON.stringify(cacheRef));
  } catch (exc) {
    Pathfinder._logError('updateCache -> error, cache: ', {
      error: Util.getMetaPreparedFromException(exc),
      cache: cacheRef,
    });
  }

  return cacheRef;
};

Pathfinder.fetch = async (prs) => {
  return new Promise((resolve) => {
    prs.callback = resolve;

    Util.fetch(prs);
  });
};

Pathfinder.fetchPfLearningPathMeta = async (prs) => {
  var goalId = prs.goalId;
  var topicId = prs.topicId;
  var chapterId = prs.chapterId;

  var output = {
    error: {
      code: '1',
      description: 'Failed performing "Pathfinder.fetchPfLearningPathMeta"'
    },
    data: null,
  };

  var fetchRes = await Pathfinder.fetch({
    context: 'Pathfinder',
    method: 'GET',
    url: Pathfinder._apiBaseUrl + '/learningpath-api/learningpath/classic?goalId=' + goalId + '&topicId=' + topicId + '&chapterId=' + chapterId,
    sendCookies: false,
    withUserSession: true,
    prepareResponseData: true,
  });

  Pathfinder._logDebug('fetchPfLearningPathMeta -> fetchRes: ', fetchRes);

  if (fetchRes.error.code === '0') {
    output.data = fetchRes.data;
  } else {
    output.error = fetchRes.error;
  }

  if (output.error.code === '1') {
    output.error = { code: '0' };
  }

  return output;
};

Pathfinder.fetchMylLearningPathNavMeta = async (prs) => {
  var output = {
    error: {
      code: '1',
      description: 'Failed performing "Pathfinder.fetchMylLearningPathNavMeta"'
    },
    data: null,
  };

  // var cache = prs.cache;
  var goalId = prs.goalId;
  var topicId = prs.topicId;
  var lessonPath = prs.lessonPath;
  var nextLessonOnly = prs.nextLessonOnly || false;

  var requestUrl = Pathfinder._mylApiBaseUrl + '/api/pathfinder/learning-path?v=2&goalId=' + goalId + '&topicId=' + topicId + '&lessonPath=' + encodeURIComponent(lessonPath);

  if (nextLessonOnly) {
    requestUrl += '&mode=next-lesson'
  }

  var fetchRes = await Pathfinder.fetch({
    context: 'Pathfinder',
    method: 'GET',
    url: requestUrl,
    sendCookies: false,
    withUserSession: true,
    prepareResponseData: true,
  });

  Pathfinder._logDebug('fetchMylLearningPathNavMeta -> fetchRes: ', fetchRes);

  if (fetchRes.error.code === '0') {
    output.data = fetchRes.data;
  } else {
    output.error = fetchRes.error;
  }

  if (output.error.code === '1') {
    output.error = { code: '0' };
  }

  return output;
};

Pathfinder.patchMylLearningPathNavMetaRefs = function (navMeta) {
  var currentLesson = navMeta.currentLesson;

  if (currentLesson !== null && currentLesson.lessonIndex !== -1) {
    navMeta.currentLesson = navMeta.topics[currentLesson.topicIndex].chapters[currentLesson.chapterIndex].lessons[currentLesson.lessonIndex];
  } else {
    navMeta.currentLesson = null;
  }

  var nextLesson = navMeta.nextLesson;

  if (nextLesson !== null && nextLesson.lessonIndex !== -1) {
    navMeta.nextLesson = navMeta.topics[nextLesson.topicIndex].chapters[nextLesson.chapterIndex].lessons[nextLesson.lessonIndex];
  } else {
    navMeta.nextLesson = null;
  }

  Pathfinder._logDebug('patchMylLearningPathNavMetaRefs -> navMeta: ', navMeta);

  return navMeta;
}

// UI

Pathfinder.uiGetIconHtml = function (params) {
  var id = params.id;
  var type = params.type;
  var size = params.size || 'auto 100%';
  var containerSize = params.containerSize || '16px 16px';
  var containerSizeChunks = containerSize.split(' ');
  var containerWidth = containerSizeChunks[0];
  var containerHeight = containerSizeChunks[1];
  var color = params.color || 'white';
  var customStyles = params.style || '';

  if (typeof type === 'undefined') {
    type = 'main';
  }

  var svgPath = '/lib/pathfinder/icon/' + type + '/' + id + '.svg';

  var svgStyles = '-webkit-mask: url(' + svgPath + ') no-repeat center;';
  svgStyles += 'mask: url(' + svgPath + ') no-repeat center;';
  svgStyles += '-webkit-mask-size: ' + size + ';';
  svgStyles += 'mask-size: ' + size + ';';
  svgStyles += 'width: ' + containerWidth + ';';
  svgStyles += 'height: ' + containerHeight + ';';
  svgStyles += customStyles;

  return '<div class="-svg-icon -svg-' + id + '-icon -color-' + color + '" style="' + svgStyles + '"></div>';
};

Pathfinder.uiFetchTemplate = async (fetchUrl) => {
  if (typeof Pathfinder._uiTemplates[fetchUrl] !== 'undefined') {
    return Pathfinder._uiTemplates[fetchUrl];
  }

  var output = {
    error: {
      code: '1',
      description: 'Failed performing "Pathfinder.uiFetchTemplate"'
    },
    data: null,
  };

  var fetchRes = await Pathfinder.fetch({
    context: 'Pathfinder',
    method: 'GET',
    url: fetchUrl,
    sendCookies: false,
    withUserSession: false,
    prepareResponseData: false,
  });

  Pathfinder._logDebug('uiFetchTemplate -> fetchUrl, fetchRes: ', {
    fetchUrl: fetchUrl,
    fetchRes: fetchRes,
  });

  if (fetchRes.error.code === '0') {
    output.data = fetchRes.dataStr;

    output.error = { code: '0' };
  } else {
    output.error = fetchRes.error;
  }

  Pathfinder._uiTemplates[fetchUrl] = output;

  return output;
};

Pathfinder.uiPrepareTemplateHtml = function (template, params) {
  var output = template;

  Object.keys(params).forEach(function (key) {
    var placeholder = '{{ ' + key + ' }}';

    output = output.replace(new RegExp(placeholder, 'g'), params[key]);
  });

  return output;
};

Pathfinder.uiUpdateMyPathNextButton = function () {
  Pathfinder.fetchMylLearningPathNavMeta({
    goalId: Pathfinder._goalMeta.uuid,
    topicId: Pathfinder._current.topicId,
    lessonPath: window.location.pathname,
    nextLessonOnly: true,
  }).then(function (res) {
    Pathfinder._logDebug('uiUpdateMyPathNextButton -> fetchMylLearningPathNavMeta -> res: ', res);

    var myPathElement = document.querySelector('.w3s-pathfinder.-my-path');
    var nextLessonBtnElement = myPathElement.querySelector('.-next-lesson-btn');

    if (res.error.code === '0') {
      var nextLesson = res.data.nextLesson;

      Pathfinder._nextLesson = nextLesson;

      Pathfinder._logDebug('uiUpdateMyPathNextButton -> fetchMylLearningPathNavMeta -> nextLesson: ', nextLesson);
    } else {
      Pathfinder._logError('uiUpdateMyPathNextButton -> fetchMylLearningPathNavMeta -> res: ', res);

      Pathfinder._nextLesson = null;

      // if (nextLessonBtnElement) {
      //   nextLessonBtnElement.classList.add('-disabled');
      // }
    }

    if (
      nextLessonBtnElement
      && nextLessonBtnElement.classList.contains('-variant-1')
    ) {
      nextLessonBtnElement.classList.add('-variant-4');
      nextLessonBtnElement.classList.remove('-variant-1');
    }
  });
};

Pathfinder.uiUpdate = function () {
  if (Pathfinder._uiProgressUpdated !== null) {
    return;
  }

  Pathfinder._uiProgressUpdated = false;

  var myPathElement = document.querySelector('.w3s-pathfinder.-my-path');

  if (!Pathfinder._userXpBumped) {
    // Pathfinder._userXp += 10;

    var xpIncrementElement = myPathElement.querySelector('.-xp .-increment');
    var xpAmountElement = myPathElement.querySelector('.-xp .-amount');
    // var progressBarElement = myPathElement.querySelector('.-progress-bar');

    if (
      !xpAmountElement
      || !xpIncrementElement
      // || !progressBarElement
    ) {
      Pathfinder._logError('uiUpdate -> related elements not found');
      return;
    }
  }

  function actualUiUpdateProgress() {
    Pathfinder._logDebug('uiUpdate -> actualUiUpdateProgress -> Pathfinder._uiProgressUpdated: ', {
      '_uiProgressUpdated': Pathfinder._uiProgressUpdated
    });

    if (Pathfinder._uiProgressUpdated) {
      return;
    }

    Pathfinder._uiProgressUpdated = true;

    var myPathElement = document.querySelector('.w3s-pathfinder.-my-path');

    Pathfinder._logDebug('uiUpdate -> actualUiUpdateProgress -> Pathfinder._userXpBumped: ', {
      '_userXpBumped': Pathfinder._userXpBumped
    });

    if (!Pathfinder._userXpBumped) {
      var xpIncrementElement = myPathElement.querySelector('.-xp .-increment');
      var xpAmountElement = myPathElement.querySelector('.-xp .-amount');

      xpAmountElement.textContent = Pathfinder._userXp;
      xpIncrementElement.classList.add('-active');

      Pathfinder._userXpBumped = true;
    }

    // var progress = Pathfinder.updateProgressOnVisitedLesson();
    // var progressPercentage = progress.percentage + '%';

    // var progressBarElement = myPathElement.querySelector('.-progress-bar');
    // var progressBarSliderElement = progressBarElement.querySelector('.-slider');

    // progressBarElement.title = progressPercentage;
    // progressBarSliderElement.title = progressPercentage;
    // progressBarSliderElement.style.width = progressPercentage;

    Pathfinder.uiUpdateMyPathNextButton();

    // nextLessonBtnElement.classList.remove('-variant-1');
    // nextLessonBtnElement.classList.add('-variant-6');
  }

  function onScrollHandler() {
    var myPathElement = document.querySelector('.w3s-pathfinder.-my-path');

    if (Util.elementIsInViewport(myPathElement)) {
      actualUiUpdateProgress();
      window.removeEventListener('scroll', onScrollHandler);
    }
  }

  if (Util.elementIsInViewport(myPathElement)) {
    actualUiUpdateProgress();
  } else {
    window.addEventListener('scroll', onScrollHandler);
  }
};

Pathfinder.uiStopClickPropagation = function (event) {
  event.stopPropagation();
}

Pathfinder.uiCloseMyPathInfoModal = function () {
  var infoWrapperElm = Util.getInnerElement(document.body, '.w3s-pathfinder.-my-path-info');

  if (infoWrapperElm) {
    // detach event listeners
    infoWrapperElm.removeEventListener('click', Pathfinder.uiStopClickPropagation);

    Util.getInnerElement(infoWrapperElm, '.-close-btn').removeEventListener('click', Pathfinder.uiCloseMyPathInfoModal);

    Util.getInnerElement(infoWrapperElm, '.-blur-overlay').removeEventListener('click', Pathfinder.uiCloseMyPathInfoModal);

    Util.getInnerElement(infoWrapperElm, '.-understand-btn').removeEventListener('click', Pathfinder.uiMyPathInfoModalHandleUnderstandBtn);

    Util.getInnerElement(infoWrapperElm, '.-dont-show-again').removeEventListener('click', Pathfinder.uiMyPathInfoModalToggleDontShowCheckbox);

    infoWrapperElm.remove();

    Pathfinder._infoModalRendered = false;
  }
};

Pathfinder.uiMyPathInfoModalHandleUnderstandBtn = function () {
  var checkboxWrapperElm = Util.getInnerElement(document.body, '.w3s-pathfinder.-my-path-info .-dont-show-again');

  if (checkboxWrapperElm) {
    if (checkboxWrapperElm.classList.contains('-checked')) {
      Cookies.set('Pathfinder.hideMyPathInfoModal', 'true', { expires: 365 });
    }
  }

  Pathfinder.uiCloseMyPathInfoModal();
};

Pathfinder.uiMyPathInfoModalToggleDontShowCheckbox = function () {
  var checkboxWrapperElm = Util.getInnerElement(document.body, '.w3s-pathfinder.-my-path-info .-dont-show-again');

  if (checkboxWrapperElm) {
    if (checkboxWrapperElm.classList.contains('-checked')) {
      checkboxWrapperElm.classList.remove('-checked');
    } else {
      checkboxWrapperElm.classList.add('-checked');
    }
  }
};

Pathfinder.uiRenderMyPathInfoModal = async () => {
  if (Pathfinder._infoModalRendered) {
    return;
  }

  Pathfinder._infoModalRendered = true;

  var templateFetchRes = await Pathfinder.uiFetchTemplate('/lib/pathfinder/html/user-authenticated/my-path-info-modal.tmpl.html');

  Pathfinder._logDebug('uiRenderMyPathInfoModal -> templateFetchRes: ', templateFetchRes);

  if (templateFetchRes.error.code !== '0') {
    Pathfinder._logError('uiRenderMyPathInfoModal -> templateFetchRes: ', templateFetchRes);
    return;
  }

  var targetElm = document.body;

  var tmpPlaceholderDiv = document.createElement('div');
  tmpPlaceholderDiv.innerHTML = templateFetchRes.data;

  targetElm.appendChild(tmpPlaceholderDiv.firstChild);

  var infoWrapperElm = Util.getInnerElement(targetElm, '.w3s-pathfinder.-my-path-info');

  if (!infoWrapperElm) {
    Pathfinder._logError('uiRenderMyPathInfoModal -> wrapper elm not found');
    return;
  }

  infoWrapperElm.addEventListener('click', Pathfinder.uiStopClickPropagation);

  Util.getInnerElement(infoWrapperElm, '.-close-btn').addEventListener('click', Pathfinder.uiCloseMyPathInfoModal);

  Util.getInnerElement(infoWrapperElm, '.-blur-overlay').addEventListener('click', Pathfinder.uiCloseMyPathInfoModal);

  Util.getInnerElement(infoWrapperElm, '.-understand-btn').addEventListener('click', Pathfinder.uiMyPathInfoModalHandleUnderstandBtn);

  Util.getInnerElement(infoWrapperElm, '.-dont-show-again').addEventListener('click', Pathfinder.uiMyPathInfoModalToggleDontShowCheckbox);
};

Pathfinder.uiRenderMyPath = async () => {
  if (Pathfinder._uiMyPathRendered) {
    return;
  }

  Pathfinder._logDebug('uiRenderMyPath -> init');

  Pathfinder._uiMyPathRendered = true;

  var templateFetchRes = await Pathfinder.uiFetchTemplate('/lib/pathfinder/html/user-authenticated/my-path.tmpl.html');

  Pathfinder._logDebug('uiRenderMyPath -> templateFetchRes: ', templateFetchRes);

  if (templateFetchRes.error.code !== '0') {
    Pathfinder._logError('uiRenderMyPath -> templateFetchRes: ', templateFetchRes);
    Pathfinder._uiMyPathRendered = false;
    return;
  }

  var tmplParams = {
    myPathUrl: Pathfinder._baseUrl + '/learningpaths',
    homeUrl: Pathfinder._baseUrl + '/learningpaths', // both my path and home are the same
    headline: Pathfinder._current.lessonFullName,
    nextLessonTitle: '',
    nextLessonBtnClass: '-variant-1',
    progressAmount: Pathfinder._userGoalProgress + '%',
    xpAmount: Pathfinder._userXp
  };

  var myPathHtml = Pathfinder.uiPrepareTemplateHtml(templateFetchRes.data, tmplParams);

  document.getElementById('user-profile-bottom-wrapper').innerHTML = myPathHtml;

  var wrapperElm = Util.getInnerElement(document.body, '.w3s-pathfinder.-my-path');

  if (document.getElementById('stickyadcontainer')) {
    wrapperElm.parentElement.classList.add('-with-ad');
  }

  var menuButtonElm = wrapperElm.querySelector('.-menu-btn');
  menuButtonElm.addEventListener('click', function () {
    if (Pathfinder._menuButtonClickProcessing) {
      return;
    }

    Pathfinder._menuButtonClickProcessing = true;

    var handleGoalNavMetaReady = function () {
      Pathfinder.uiRenderMenuModal();

      Pathfinder._menuButtonClickProcessing = false;
    };

    if (Pathfinder._goalNavMeta === null) {
      Pathfinder.fetchMylLearningPathNavMeta({
        goalId: Pathfinder._goalMeta.uuid,
        topicId: Pathfinder._current.topicId,
        lessonPath: window.location.pathname,
        nextLessonOnly: false,
      }).then(function (res) {
        Pathfinder._logDebug('menuButtonElm -> click -> fetchMylLearningPathNavMeta -> res: ', res);

        if (res.error.code === '0') {
          Pathfinder._goalNavMeta = Pathfinder.patchMylLearningPathNavMetaRefs(res.data.navMetaLite);

          Pathfinder._logDebug('menuButtonElm -> click -> fetchMylLearningPathNavMeta -> _goalNavMeta: ', Pathfinder._goalNavMeta);

          handleGoalNavMetaReady();
        } else {
          Pathfinder._logError('menuButtonElm -> click -> fetchMylLearningPathNavMeta -> res: ', res);
        }
      });
    } else {
      handleGoalNavMetaReady();
    }
  });

  var nextLessonButtonElm = wrapperElm.querySelector('.-next-lesson-btn');
  nextLessonButtonElm.addEventListener('click', function () {
    if (Pathfinder._nextLessonButtonClicked) {
      return;
    }

    Pathfinder._nextLessonButtonClicked = true;

    // next lesson is fetched in background when pf component enters the viewport, we 
    Util.objFieldOnSetCallback({
      scopeRef: Pathfinder,
      fieldName: '_nextLesson',
      callback: function (res) {
        if (res.error.code === '0') {
          if (Pathfinder._nextLesson === null) { // no more lessons to visit
            window.location.href = Pathfinder._baseUrl + '/learningpaths';
          } else if (typeof Pathfinder._nextLesson.path !== 'undefined') {
            window.location.href = '/' + Pathfinder._nextLesson.path;
          } else {
            Pathfinder._logError('nextLessonButtonElm -> click -> Pathfinder._nextLesson: ', Pathfinder._nextLesson);
          }
        } else {
          Pathfinder._logError('nextLessonButtonElm -> click -> res: ', res);
        }
      }
    });
  });

  if (
    Pathfinder._currentLessonFinishedOnInit === false
  ) {
    MyLearning.lessonFinishedListeners['Pathfinder -> my path'] = function () {
      Pathfinder.uiUpdate();
    };

    if (MyLearning.lessonFinished) {
      Pathfinder.uiUpdate();
    }
  } else {
    Pathfinder.uiUpdate();
  }

  // cache menu template
  await Pathfinder.uiFetchTemplate('/lib/pathfinder/html/user-authenticated/my-path-menu-modal.tmpl.html');
};

Pathfinder.uiRenderSkills = function (wrapperElm, skills) {
  var outerDiv = document.createElement('div');
  outerDiv.textContent = 'Required skills: ';

  for (var key in skills) {
    if (skills.hasOwnProperty(key)) {
      var span = document.createElement('span');
      span.style.cssText = 'display: inline-block; padding: 2px 5px; margin: 3px; font-size: 10px;';
      span.className = 'w3-blue w3-round-small w3-border';
      span.textContent = key + ': ' + skills[key].level;
      outerDiv.appendChild(span);
    }
  }

  wrapperElm.appendChild(outerDiv);
};

Pathfinder.uiCloseMenuModal = function () {
  var menuWrapperElm = Util.getInnerElement(document.body, '.w3s-pathfinder.-my-path-menu');

  if (menuWrapperElm) {
    // detach event listeners
    menuWrapperElm.removeEventListener('click', Pathfinder.uiStopClickPropagation);

    Util.getInnerElement(menuWrapperElm, '.-close-btn').removeEventListener('click', Pathfinder.uiCloseMenuModal);

    Util.getInnerElement(menuWrapperElm, '.-blur-overlay').removeEventListener('click', Pathfinder.uiCloseMenuModal);

    menuWrapperElm.remove();

    Pathfinder._menuModalRendered = false;
  }
};

Pathfinder.uiRenderMenuWrapper = function (targetElm, template) {
  if (Pathfinder._menuModalRendered) {
    return Util.getInnerElement(targetElm, '.w3s-pathfinder.-my-path-menu');
  }

  Pathfinder._menuModalRendered = true;

  if (typeof targetElm === 'undefined') {
    targetElm = document.body;
  }

  var tmpPlaceholderDiv = document.createElement('div');
  tmpPlaceholderDiv.innerHTML = template;

  targetElm.appendChild(tmpPlaceholderDiv.firstChild);

  var menuWrapperElm = Util.getInnerElement(targetElm, '.w3s-pathfinder.-my-path-menu');

  if (!menuWrapperElm) {
    Pathfinder._logError('uiRenderMenuWrapper -> wrapper elm not found');
    return;
  }

  menuWrapperElm.addEventListener('click', Pathfinder.uiStopClickPropagation);

  Util.getInnerElement(menuWrapperElm, '.-close-btn').addEventListener('click', Pathfinder.uiCloseMenuModal);

  Util.getInnerElement(menuWrapperElm, '.-blur-overlay').addEventListener('click', Pathfinder.uiCloseMenuModal);

  return menuWrapperElm;
};

Pathfinder.uiRenderMenuModal = function () {
  var menuModalTemplateUrl = '/lib/pathfinder/html/user-authenticated/my-path-menu-modal.tmpl.html';

  if (
    typeof Pathfinder._uiTemplates[menuModalTemplateUrl] === 'undefined'
    || Pathfinder._uiTemplates[menuModalTemplateUrl].error.code !== '0'
  ) {
    Pathfinder._logError('uiRenderMenuModal -> menuModalTemplateUrl cache error: ', Pathfinder._uiTemplates[menuModalTemplateUrl]);
    return;
  }

  var menuWrappermElm = Pathfinder.uiRenderMenuWrapper(window.body, Pathfinder._uiTemplates[menuModalTemplateUrl].data);

  if (!menuWrappermElm) {
    Pathfinder._logError('uiRenderMenuModal -> menuWrappermElm not found');
    return;
  }

  var navMenuInnerWrapperElm = Util.getInnerElement(menuWrappermElm, '.-scroll-box');

  function createLessonListItem(lesson) {
    var lessonItem = document.createElement('div');
    lessonItem.className = '-lesson';
    var lessonLink = document.createElement('a');
    // lessonLink.style.cssText = 'width: 100%; padding: 4px !important;';
    lessonLink.href = "/" + lesson.path;
    lessonLink.innerHTML = Pathfinder.uiGetIconHtml({ id: 'check', containerSize: '24px 24px', style: 'top: -2px; margin-right: 10px; margin-bottom: -5px;' }) + Pathfinder.uiGetIconHtml({ id: 'lesson', style: 'bottom: -2px; margin-right: 11px;' }) + lesson.name;

    var lessonLinkClassName = '-link w3-button w3-small';

    if (lesson.current) {
      lessonLinkClassName += ' -current';
    }

    if (lesson.visited) {
      lessonLinkClassName += ' -visited';
    }

    if (lesson.goalTarget) {
      lessonLinkClassName += ' -goal-target';

      if (typeof lesson.nextGoalTarget !== 'undefined' && lesson.nextGoalTarget) {
        lessonLinkClassName += ' -next-goal-target';
      }
    } else {
      lessonLinkClassName += ' -not-goal-target';
    }

    lessonLink.className = lessonLinkClassName;

    // listItem.style.cssText = 'border-bottom: 1px solid darkgrey;';
    // listItem.style.cssText = 'position: relative; margin-bottom: 2px;';

    lessonItem.appendChild(lessonLink);

    return lessonItem;
  }

  function createChapterListItem(topic, chapter) {
    var chapterWrapper = document.createElement('div');
    chapterWrapper.className = '-sub-list';
    // sublist.style.cssText = 'padding: 0 0 10px 20px; position: relative;';
    var chapterName = document.createElement('div');
    chapterName.className = '-name';
    // chapterName.style.cssText = 'font-weight: bold; position: relative; left: -20px; padding-bottom: 10px;';
    chapterName.innerHTML = Pathfinder.uiGetIconHtml({ id: topic.uuid, type: 'skill', containerSize: '20px 20px', style: 'bottom: -3px; margin-right: 7px;' }) + topic.name + ' - ' + chapter.name;
    chapterWrapper.appendChild(chapterName);

    chapter.lessons.forEach(function (lesson) {
      chapterWrapper.appendChild(createLessonListItem(lesson));
    });

    return chapterWrapper;
  }

  function createTopicListItem(topic) {
    var topicWrapper = document.createElement('div');
    topicWrapper.className = '-list';
    // topicItem.style.cssText = 'padding: 0 0 0 20px; position: relative;';
    var topicName = document.createElement('div');
    topicName.className = '-name';
    // topicName.style.cssText = 'font-weight: bold; position: relative; left: -20px;';
    topicName.innerHTML = Pathfinder.uiGetIconHtml({ id: topic.uuid, type: 'skill', containerSize: '20px 20px', style: 'bottom: -3px; margin-right: 7px;' }) + topic.name;
    topicWrapper.appendChild(topicName);

    topic.chapters.forEach(function (chapter) {
      topicWrapper.appendChild(createChapterListItem(topic, chapter));
    });

    return topicWrapper;
  }

  var navMenu = document.createElement('div');
  navMenu.style.cssText = '';

  var topics = Pathfinder._goalNavMeta.topics;

  topics.forEach(function (topic) {
    navMenu.appendChild(createTopicListItem(topic));
  });

  navMenuInnerWrapperElm.appendChild(navMenu);

  var activeLink = navMenuInnerWrapperElm.querySelector('.-link.-current');

  if (activeLink) {
    activeLink.scrollIntoView();
  }
};
