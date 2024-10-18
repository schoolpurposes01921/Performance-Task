/*
  MyLearning script
*/

window.MyLearning = {
  version: '1.0.29',
  progressTrackingVersion: '1.0.2', // WARNING! Bump with care as it invalidates user's progress
  _isIframed: null,
  _iframeOriginIndex: -1,
  _iframeSendBackLocationAndLessonMeta: false,
  _iframeParentSentMessages: {},
  pages_read_count: 0,
  total_pages_count: 0,
  user_progress_collected: false,
  _w3sBaseUrl: 'https://www.w3schools.com',
  _profileBaseUrl: 'https://profile.w3schools.com',
  _linkTargetAttr: '',
  _baseUrl: 'https://my-learning.w3schools.com',
  _apiBaseUrl: 'https://myl-api.w3schools.com',
  lessonInitialized: null,
  lessonFinished: null,
  lessonFinishedListeners: {}, // key: listenerId, value: callback
  localStorageAmount: 5242880, // 5MB
  localStorageUsedAmount: 0,
  localStorageTrackingAmount: 3145728, // 3MB
  localStorageTrackingMaxCount: 1000,
  // pageIsHome: boolean,
  // topicId: string,
  // chapterId: string,
  // pageType: string,
  // lessonId: string,
  // lessonFullName: string,
  // lessonFinishedOnInit: boolean,
  // lessonMeta: any,
  ps: '',
  _uiTemplates: {},
  _uiSidenavPatchLayoutDebouncer: null,
};

MyLearning._isDebugMode = function () {
  return Util.isDebugMode('MyLearning');
}

MyLearning._logDebug = function (message, data, logRawData) {
  return Util.logDebug('MyLearning', message, data, logRawData);
}

MyLearning._logWarning = function (message, data, logRawData) {
  return Util.logWarning('MyLearning', message, data, logRawData);
}

MyLearning._logError = function (message, data, logRawData) {
  return Util.logError('MyLearning', message, data, logRawData);
}

MyLearning.isIframed = function () {
  if (MyLearning._isIframed !== null) {
    return MyLearning._isIframed;
  }

  try {
    MyLearning._isIframed = window.self !== window.top;
  } catch (e) {
    MyLearning._isIframed = true;
  }

  return MyLearning._isIframed;
}

MyLearning._iframeInit = function () {
  MyLearning._logDebug('_iframeInit');

  window.open = function (url, target, features) {
    MyLearning._logDebug('window.open -> url, target, features: ', {
      url: url,
      target: target,
      features: features,
    });

    // if url on same domain, open in parent window
    if (url.indexOf('https://www.w3schools.com') === 0) {
      window.location.href = url;
    } else {
      window.parent.postMessage({
        'action': 'OPEN_URL',
        'data': {
          'url': url,
          'target': target,
          'features': features,
        },
      }, '*');
    }
  };

  window.addEventListener('message', function (message) {
    if (MyLearning._isDebugMode()) {
      console.log(message); // raw
    }
    MyLearning._logDebug('message: ', message);

    var whitelistedOrigins = [
      'https://profile.w3schools.com',
      'https://pathfinder.w3schools.com',
      'https://my-learning.w3schools.com',
      'https://spaces.w3schools.com',
      'https://billing.w3schools.com',
      'https://campus.w3schools.com',
      // #dev 'https://pathfinder-local.w3schools.com:3000',
    ];

    var iframeOriginIndex = whitelistedOrigins.indexOf(message.origin);

    if (iframeOriginIndex === -1) {
      // MyLearning._logWarning('Invalid iframe origin: ', message.origin);
      // MyLearning._isIframed = false;
      return;
    }

    MyLearning._iframeOriginIndex = iframeOriginIndex;

    var action = '';
    var data = null;

    if (typeof message.data.type !== 'undefined') {
      action = message.data.type;
      data = message.data.value;
    } else if (typeof message.data.action !== 'undefined') {
      action = message.data.action;
      data = message.data.data;
    }

    MyLearning._logDebug('message -> action, data: ', {
      action: action,
      data: data,
    });

    if (action === 'INIT') {
      MyLearning._iframeInitHandle(data);
    } else if (action === 'FETCH_LOCATION') {
      MyLearning._iframeFetchLocationHandle(data);
    } else if (action === 'FETCH_NEXT_PREV_BTNS') {
      MyLearning._iframeFetchNextPrevBtns(data);
    } else if (action === 'UPDATE_NEXT_PREV_BTNS') {
      MyLearning._iframeUpdateNextPrevBtns(data);
    }
  });
}

MyLearning._iframeFetchNextPrevBtns = function (data) {
  MyLearning._logDebug('_iframeFetchNextPrevBtns -> data: ', data);

  var nextPrevWrapperElms = document.querySelectorAll('.nextprev');

  var nextPrevBtns = {
    "top": {
      "prev": { "href": "", "text": "" },
      "next": { "href": "", "text": "" }
    },
    "bottom": {
      "prev": { "href": "", "text": "" },
      "next": { "href": "", "text": "" }
    }
  };

  if (nextPrevWrapperElms.length) {
    function fetchPrevNextBtns(btnGroup, links) {
      if (links.length > 0) {
        btnGroup.prev.href = links[0].getAttribute('href') || "";
        btnGroup.prev.text = links[0].innerText || "";
      }
      if (links.length > 1) {
        btnGroup.next.href = links[1].getAttribute('href') || "";
        btnGroup.next.text = links[1].innerText || "";
      }
    }

    var topNextPrevBtnElms = nextPrevWrapperElms[0].querySelectorAll('a');
    fetchPrevNextBtns(nextPrevBtns.top, topNextPrevBtnElms);

    if (nextPrevWrapperElms.length > 1) {
      var bottomNextPrevBtnElms = nextPrevWrapperElms[1].querySelectorAll('a');
      fetchPrevNextBtns(nextPrevBtns.bottom, bottomNextPrevBtnElms);
    }
  }

  window.parent.postMessage({
    'action': 'SHARING_NEXT_PREV_BTNS_DATA',
    'data': {
      'nextPrevBtns': nextPrevBtns
    }
  }, '*');
};

MyLearning._iframeUpdateNextPrevBtns = function (data) {
  MyLearning._logDebug('_iframeUpdateNextPrevBtns -> data: ', data);

  var nextPrevWrapperElms = document.querySelectorAll('.nextprev');

  if (nextPrevWrapperElms.length) {
    var nextPrevBtns = {
      "top": {},
      "bottom": {}
    };

    var setByShortcut = false;

    if (typeof data.prev !== 'undefined') {
      nextPrevBtns.top.prev = data.prev;
      nextPrevBtns.bottom.prev = data.prev;
      setByShortcut = true;
    }

    if (typeof data.next !== 'undefined') {
      nextPrevBtns.top.next = data.next;
      nextPrevBtns.bottom.next = data.next;
      setByShortcut = true;
    }

    if (!setByShortcut) {
      nextPrevBtns = data;
    }

    function isPrevNextUrlAllowed(url) {
      return url.indexOf('https://www.w3schools.com') === 0 || url.indexOf('/') === 0;
    }

    function setPrevNextBtns(btnGroup, links) {
      if (links.length > 0 && typeof btnGroup.prev !== 'undefined') {
        if (btnGroup.prev.href && isPrevNextUrlAllowed(btnGroup.prev.href)) {
          links[0].setAttribute('href', btnGroup.prev.href);
        }

        if (btnGroup.prev.text) {
          links[0].innerText = btnGroup.prev.text;
        }
      }

      if (links.length > 1 && typeof btnGroup.next !== 'undefined') {
        if (btnGroup.next.href && isPrevNextUrlAllowed(btnGroup.next.href)) {
          links[1].setAttribute('href', btnGroup.next.href);
        }

        if (btnGroup.next.text) {
          links[1].innerText = btnGroup.next.text;
        }
      }
    }

    if (typeof nextPrevBtns.top !== 'undefined') {
      var topNextPrevBtnElms = nextPrevWrapperElms[0].querySelectorAll('a');
      setPrevNextBtns(nextPrevBtns.top, topNextPrevBtnElms);
    }

    if (typeof nextPrevBtns.bottom !== 'undefined') {
      var bottomNextPrevBtnElms = nextPrevWrapperElms[1].querySelectorAll('a');
      setPrevNextBtns(nextPrevBtns.bottom, bottomNextPrevBtnElms);
    }
  }
};

MyLearning._iframeInitHandle = function (data) {
  MyLearning._logDebug('_iframeInitHandle -> data: ', { data: data });

  if (typeof data !== 'undefined') {
    if (
      typeof data.sendBackLocationAndLessonMeta !== 'undefined'
      && data.sendBackLocationAndLessonMeta === true
    ) {
      MyLearning._iframeSendBackLocationAndLessonMeta = true;

      MyLearning._iframeParentPushLocation();

      if (MyLearning.lessonFinished) {
        MyLearning._iframeParentPushLessonMeta('COMPLETED_LESSON');
      } else if (MyLearning.lessonInitialized) {
        MyLearning._iframeParentPushLessonMeta('INITIALIZED_LESSON');
      }
    }
  }
}

MyLearning._iframeParentPushLocation = function () {
  MyLearning._logDebug('_iframeParentPushLocation');

  window.parent.postMessage({
    'action': 'SHARING_LOCATION',
    'data': {
      'location': {
        'href': window.location.href,
        'hostname': window.location.hostname,
        'pathname': window.location.pathname,
      }
    }
  }, '*');
}

MyLearning._iframeParentPushLessonMeta = function (action) {
  MyLearning._logDebug('_iframeParentPushLessonMeta -> action: ', {
    action: action,
    debounce: MyLearning._iframeParentSentMessages[action],
  });

  if (typeof MyLearning._iframeParentSentMessages[action] !== 'undefined') {
    return;
  }

  MyLearning._iframeParentSentMessages[action] = true;

  var message = {
    'action': action,
    'data': {
      'lessonMeta': MyLearning.lessonMeta,
      'location': {
        'href': window.location.href,
        'hostname': window.location.hostname,
        'pathname': window.location.pathname,
      }
    }
  };

  MyLearning._logDebug('_iframeParentPushLessonMeta -> message: ', message);

  window.parent.postMessage(message, '*');
}

MyLearning._iframeFetchLocationHandle = function (prs) {
  MyLearning._logDebug('_iframeFetchLocationHandle -> prs: ', prs);

  MyLearning._iframeParentPushLocation();
}

MyLearning._apiNameToPathMap = {
  'api.meta': '/api/classic/get-set-topic-progress',
  'api.exercise.get': '/api/classic/get-exercises-progress',
  'api.exercise.set': '/api/classic/set-exercises-progress',
  'api.quiz.set_score': '/api/classic/set-quiz-progress'
}

MyLearning.getUrl = function (apiName) {
  this._logDebug('getUrl: ', apiName);

  return MyLearning._apiBaseUrl + this._apiNameToPathMap[apiName];
}
// << classic

// < common

MyLearning.uiFetchTemplate = function (fetchUrl, callback) {
  if (typeof MyLearning._uiTemplates[fetchUrl] !== 'undefined') {
    return MyLearning._uiTemplates[fetchUrl];
  }

  Util.fetch({
    context: 'MyLearning',
    method: 'GET',
    url: fetchUrl,
    sendCookies: false,
    withUserSession: false,
    prepareResponseData: false,
    callback: function (fetchRes) {
      MyLearning._logDebug('uiFetchTemplate -> fetchUrl, fetchRes: ', {
        fetchUrl: fetchUrl,
        fetchRes: fetchRes,
      });

      var output = {
        error: {
          code: '1',
          description: 'Failed performing "MyLearning.uiFetchTemplate"'
        },
        data: null,
      };

      if (fetchRes.error.code === '0') {
        output.data = fetchRes.dataStr;

        output.error = { code: '0' };
      } else {
        output.error = fetchRes.error;
      }

      MyLearning._uiTemplates[fetchUrl] = output;

      if (typeof callback !== 'undefined') {
        callback(output);
      }
    }
  });
};

MyLearning.uiPrepareTemplateHtml = function (template, params) {
  var output = template;

  Object.keys(params).forEach(function (key) {
    var placeholder = '{{ ' + key + ' }}';

    output = output.replace(new RegExp(placeholder, 'g'), params[key]);
  });

  return output;
};

MyLearning.makePostRequest = function (url, data, callback) {
  Util.fetch({
    context: 'MyLearning -> makePostRequest',
    method: 'POST',
    url: url,
    data: data,
    callback: callback,
    withUserSession: true,
    handleUserSessionRedirection: true, // TODO: (mid) revisit this flag
    prepareResponseData: false,
    headers: {
      'Content-Type': 'application/x-www-form-urlencoded'
    }
  });
};

MyLearning.getCircleMeta = function (xx, yy, r, aD) {
  var aR = (aD - 90) * Math.PI / 180.0;
  return {
    x: xx + (r * Math.cos(aR)),
    y: yy + (r * Math.sin(aR))
  };
};

MyLearning.getProfileIconCirclesRendered = function (x, y, r, sa, ea) {
  var s = MyLearning.getCircleMeta(x, y, r, ea);
  var e = MyLearning.getCircleMeta(x, y, r, sa);
  var f = ea - sa <= 180 ? "0" : "1";
  return ["M", s.x, s.y, "A", r, r, 0, f, 0, e.x, e.y].join(" ");
};

MyLearning.loadUser = function (context, callback) {
  this._logDebug('loadUser -> args: ', [context]);


  UserSession.processUserSession({
    context: context + ' -> MyLearning -> loadUser',
    handleRedirection: true,
    callback: function (userSessionVerificationRes) {
      if (context === 'footer') {
        MyLearning._footerLoadUser();
      }

      if (!TopNavBar.configured) {
        TopNavBar.postInitConfig({
          env: 'classic',
          loggedIn: UserSession.loggedIn === true,
          subscriptionPlan: UserSession.getUserSubscriptionPlan(),
          location: window.location
        });
      }

      if (UserSession.loggedIn === true) {
        Util.attachScript({
          id: 'pathfinder-script',
          path: '/lib/pathfinder/main.js',
          entityName: 'Pathfinder',
          callback: function (res) {
            if (res.error.code === '0') {
              Pathfinder.init();
            } else {
              console.error('Failed loading pathfinder util -> error: ', res.error);
            }
          }
        });
      }

      if (typeof callback !== 'undefined') {
        callback(userSessionVerificationRes);
      }
    }
  });
}

MyLearning.getStrWithPrefixRemoved = function (str, subStr) {
  const extractedChunk = str.slice(0, subStr.length);

  if (extractedChunk === subStr) {
    return str.slice(subStr.length);
  }

  return str;
}
// > common

// < footer
MyLearning._footerLoadUser = function () { // on lessons this is the first request, on quiz the only one
  this._logDebug('_footerLoadUser');

  if (!UserSession.loggedIn) {
    return;
  }

  var urlPath = window.location.pathname,
    urlPathWlo = this.getStrWithPrefixRemoved(urlPath, '/'), // wlo - without leading slash
    isQuizPage = false;

  if (urlPathWlo.indexOf('quiztest/quiztest') === 0) {
    isQuizPage = true;

    var pathMetaStr = sessionStorage.getItem(urlPath);

    if (pathMetaStr !== null) {
      var pathMeta = JSON.parse(pathMetaStr);

      if (pathMeta.isQuizPage) {
        this.renderUserProgress(pathMeta.reqRes.type, pathMeta.reqRes.raw);
        return;
      }
    }
  }

  // show the user active session first and update the progress when we have it on hands
  this.renderUserProgress('T', 'T{"a":0,"b":0}'); // T - unused state, stands for "Temporary/Transitory"

  var x, y, pos, foldername, filename, typ, cc, pathname = window.location.pathname;
  if (pathname.substr(0, 1) == "/") { pathname = pathname.substr(1); }
  pos = pathname.indexOf("/");
  foldername = pathname.substr(0, pos);
  if (pathname.indexOf("pandas") > -1) { foldername = "python/pandas"; }
  if (pathname.indexOf("numpy") > -1) { foldername = "python/numpy"; }
  if (pathname.indexOf("scipy") > -1) { foldername = "python/scipy"; }
  filename = pathname.substr(pos + 1);
  typ = foldername;
  var v = '1';
  if (foldername == "quiztest") {
    cc = window.location.href;
    pos = cc.indexOf("qtest=");
    typ = cc.substr(pos + 6);
  } else {
    v = '2';
  }

  var reqDataStr = "v=" + v + "&a=" + foldername + "&b=" + filename + "&c=" + typ + "&d=0&p=" + encodeURIComponent(window.location.pathname);

  // initial request to get the user progress
  MyLearning.makePostRequest(
    MyLearning.getUrl('api.meta'),
    reqDataStr,
    function (reqRes) {
      if (
        reqRes.error.code === '0'
        && reqRes.status === 200
      ) {
        var y = reqRes.dataStr;
        var x = y.substr(0, 1);

        MyLearning._logDebug('_footerLoadUser -> req_res -> x: ', x);

        if (x == "F" || x == "G" || x == "H" || x == "I" || x == "J" || x == "K" || x == "L" || x == "M" || x == "Q") {
          var reqResDataParseRes = Util.parseJson(y.substr(1));

          if (reqResDataParseRes.error.code === '0') {
            var reqResData = reqResDataParseRes.data;

            MyLearning._logDebug('_footerLoadUser -> reqResData: ', reqResData);

            if (typeof reqResData.topicId !== 'undefined') {
              MyLearning.topicId = reqResData.topicId;
            }

            if (typeof reqResData.chapterId !== 'undefined') {
              MyLearning.chapterId = reqResData.chapterId;
            }

            if (typeof reqResData.isHome !== 'undefined') {
              MyLearning.pageIsHome = reqResData.isHome;

              var currentPath = Util.removeStrPrefix(window.location.pathname, '/');

              MyLearning._uiSidenavFlagCompletedLinks([currentPath]);
            }

            if (typeof reqResData.type !== 'undefined') {
              MyLearning.pageType = reqResData.type;
            }

            if (typeof reqResData.lessonId !== 'undefined') {
              MyLearning.lessonId = reqResData.lessonId;
            }

            if (typeof reqResData.lessonFullName !== 'undefined') {
              MyLearning.lessonFullName = reqResData.lessonFullName;
            }

            if (typeof reqResData.ps !== 'undefined') {
              MyLearning.ps = reqResData.ps;
            }

            MyLearning.lessonFinishedOnInit = (x === 'I');

            MyLearning.lessonMeta = reqResData;
            MyLearning.lessonMeta.completedOnInit = MyLearning.lessonFinishedOnInit;

            MyLearning.lessonInitialized = true;

            MyLearning._logDebug('Lesson initialized', {
              'MyLearning.isIframed()': MyLearning.isIframed(),
              'MyLearning._iframeSendBackLocationAndLessonMeta': MyLearning._iframeSendBackLocationAndLessonMeta,
              'MyLearning.lessonMeta': MyLearning.lessonMeta,
            });

            if (MyLearning.isIframed() && MyLearning._iframeSendBackLocationAndLessonMeta) {
              MyLearning._iframeParentPushLessonMeta('INITIALIZED_LESSON');
            }
          }

          MyLearning.user_progress_collected = true;
          MyLearning.renderUserProgress(x, y); // this one sets the scroll event

          if (isQuizPage) {
            sessionStorage.setItem(urlPath, JSON.stringify({
              'isQuizPage': true,
              'reqRes': {
                'type': x,
                'raw': y
              }
            }));
          }
        }
      }
    }
  );
}

MyLearning._checkIfGotToTheBottomOfThePageDebouncer = null;

MyLearning._checkIfGotToTheBottomOfThePage = function () {
  clearTimeout(MyLearning._checkIfGotToTheBottomOfThePageDebouncer);

  MyLearning._checkIfGotToTheBottomOfThePageDebouncer = setTimeout(function () {
    var userProfileBottomWrapperElm = document.querySelector('.user-profile-bottom-wrapper');

    var reachedUserProfileBottomWrapperElm = Util.elementIsInViewport(userProfileBottomWrapperElm) || Util.elementIsAboveViewport(userProfileBottomWrapperElm);

    if (reachedUserProfileBottomWrapperElm) {
      MyLearning._logDebug('_checkIfGotToTheBottomOfThePage: ', true);

      window.removeEventListener('scroll', MyLearning._checkIfGotToTheBottomOfThePage);

      if (UserSession.loggedIn) {
        MyLearning._handleFinishedPageAuthenticated();
      } else {
        MyLearning._handleFinishedPageAnonymous();
      }
    }
  }, 100);
}

MyLearning._findInnerElements = function (parentElement, queryStr, callback) {
  var output = [];

  var hasCallback = typeof callback !== 'undefined';

  var elements = parentElement.querySelectorAll(queryStr);

  for (var index = 0; index < elements.length; index++) {
    output.push(elements[index]);

    if (hasCallback) {
      callback(elements[index], index);
    }
  }

  return output;
};

MyLearning._loopArray = function (arr, callback) {
  for (var index = 0; index < arr.length; index++) {
    callback(arr[index], index);
  }
};

MyLearning.renderUserProgress = function (cc, obj) {
  this._logDebug('renderUserProgress -> args: ', [cc, obj]);

  var x,
    degrees = 0,
    // color1,
    // color2,
    jsonobj;

  if (document.getElementById("top-nav-bar")) {
    if (cc == "I" || cc == "J" || cc == "H" || cc == "G" || cc == "O" || cc == "Q") {
      jsonobj = JSON.parse(obj.substr(1));
      this.pages_read_count = jsonobj.b;
      this.total_pages_count = jsonobj.a;
      x = Math.round((this.pages_read_count / this.total_pages_count) * 100);
      degrees = x * 3.6;
      if (degrees > 359) degrees = 359.99;
    }

    // if (cc == "Q") {
    //   color1 = "rgba(44, 156, 202, 0.1)";
    //   color2 = "rgba(44, 156, 202, 1)";
    // } else {
    //   color1 = "rgba(4, 170, 109, 0.1)";
    //   color2 = "rgba(4, 170, 109, 1)";
    // }

    if (cc == "I" || cc == "J" || cc == "H" || cc == "G") {
      window.addEventListener('scroll', MyLearning._checkIfGotToTheBottomOfThePage);

      MyLearning._checkIfGotToTheBottomOfThePage();
    }

    // MyLearning._findInnerElements(document, '.user-progress-circle1', function (userProgressCircle1Elm) {
    //   if (cc == "Q") {
    //     userProgressCircle1Elm.setAttribute("stroke", "rgba(44, 156, 202, 0.1)");
    //   } else {
    //     userProgressCircle1Elm.setAttribute("stroke", "rgba(4, 170, 109, 0.1)");
    //   }

    //   // if (cc != "L" && cc != "F") {
    //   //   userProgressCircle1Elm.setAttribute("d", MyLearning.getProfileIconCirclesRendered(26, 35, 20, 0, 359.99));
    //   // }
    // });

    MyLearning._findInnerElements(document, '.user-progress-circle2', function (userProgressCircle2Elm) {
      // if (cc == "Q") {
      //   userProgressCircle2Elm.setAttribute("stroke", "rgba(44, 156, 202, 1)");
      // } else {
      //   userProgressCircle2Elm.setAttribute("stroke", "rgba(4, 170, 109, 1)");
      // }

      if (cc != "L" && cc != "F") {
        userProgressCircle2Elm.setAttribute("d", MyLearning.getProfileIconCirclesRendered(26, 35, 20, 0, degrees));
      }
    });

    if (cc == "Q") {
      if (degrees == 359.99) {
        MyLearning._findInnerElements(document, '.user-progress-star', function (userProgressStarElm) {
          userProgressStarElm.classList.add("up-active");
        });
      }
    }
  }

  // ga('send', 'event', 'user', 'login');
}

MyLearning._handleFinishedPageAuthenticated = function () {
  this._logDebug('_handleFinishedPageAuthenticated');

  if (!UserSession.loggedIn || !this.user_progress_collected) {
    this._logDebug('_handleFinishedPageAuthenticated -> jumping out');
    return;
  }

  var x, y, pos, foldername, filename, typ, pathname = window.location.pathname;
  if (pathname.substr(0, 1) == "/") { pathname = pathname.substr(1); }
  pos = pathname.indexOf("/");
  foldername = pathname.substr(0, pos);
  if (pathname.indexOf("pandas") > -1) { foldername = "python/pandas"; }
  if (pathname.indexOf("numpy") > -1) { foldername = "python/numpy"; }
  if (pathname.indexOf("scipy") > -1) { foldername = "python/scipy"; }
  filename = pathname.substr(pos + 1);
  typ = foldername;

  var reqDataStr = "a=" + foldername + "&b=" + filename + "&c=" + typ + "&d=1&p=" + encodeURIComponent(window.location.pathname) + "&ps=" + MyLearning.ps;

  // subsequent request to update the user progress
  MyLearning.makePostRequest(
    MyLearning.getUrl('api.meta'),
    reqDataStr,
    function (reqRes) {
      if (
        reqRes.error.code === '0'
        && reqRes.status === 200
      ) {
        var y = reqRes.dataStr;
        var x = y.substr(0, 1);

        if (x == "O") {
          if (MyLearning.pageType === 'lesson') {
            MyLearning.registerPointForFinishedLesson(x);
          }

          var currentPath = Util.removeStrPrefix(window.location.pathname, '/');

          MyLearning._uiSidenavFlagCompletedLinks([currentPath]);
        }
      }
    }
  );
}

MyLearning.registerPointForFinishedLesson = function () {
  MyLearning._logDebug('registerPointForFinishedLesson');

  if (document.getElementById("top-nav-bar")) {
    if (
      typeof MyLearning.lessonFinishedOnInit === 'undefined'
      || MyLearning.lessonFinishedOnInit === false
    ) {
      this.pages_read_count++;

      MyLearning.renderProgress((this.pages_read_count / this.total_pages_count) * 100, true);
    }

    MyLearning.lessonFinished = true;

    if (MyLearning.isIframed() && MyLearning._iframeSendBackLocationAndLessonMeta) {
      MyLearning._iframeParentPushLessonMeta('COMPLETED_LESSON');
    }

    MyLearning.notifyListenersOnFinishedLesson();
  }
}

MyLearning.notifyListenersOnFinishedLesson = function () {
  MyLearning._logDebug('notifyListenersOnFinishedLesson');

  for (var listenerId in MyLearning.lessonFinishedListeners) {
    MyLearning.lessonFinishedListeners[listenerId]();
  }
};

MyLearning.renderProgress = function (percentPoints, isSinglePoint) {
  MyLearning._logDebug('renderProgress -> percentPoints: ', percentPoints);

  if (typeof isSinglePoint === 'undefined') {
    isSinglePoint = false;
  }

  var degrees = 0,
    completed = false;

  degrees = Math.round(percentPoints) * 3.6;

  if (degrees > 359) {
    degrees = 359.99;
    completed = true;
  };

  MyLearning._findInnerElements(document, '.user-progress-circle2', function (userProgressCircle2Elm) {
    userProgressCircle2Elm.setAttribute('d', MyLearning.getProfileIconCirclesRendered(26, 35, 20, 0, degrees));
  });

  if (completed) {
    MyLearning._findInnerElements(document, '.user-progress-star', function (userProgressStarElm) {
      userProgressStarElm.classList.add('up-active');
    });
  } else {
    if (isSinglePoint) {
      MyLearning._findInnerElements(document, '.user-progress-point', function (userProgressPointElm) {
        userProgressPointElm.classList.add('up-active');
      });
    }
  }
}

MyLearning.removeFooterProfileBtn = function () {
  document.getElementById('user-profile-bottom-wrapper').remove();
}

MyLearning.showQuizScoreProfileBtn = function () {
  document.getElementById('user-profile-quiz-score-wrapper').classList.remove('w3-hide');
}
// > footer
// >> classic


MyLearning._uiSidenavPatchLayout = function (event) {
  MyLearning._logDebug('_uiSidenavPatchLayout -> event: ', {
    event: event
  });

  clearTimeout(MyLearning._uiSidenavPatchLayoutDebouncer);

  var timeoutDelay = event === true ? 0 : 500;

  MyLearning._uiSidenavPatchLayoutDebouncer = setTimeout(function () {
    var sidenavElm = document.getElementById('leftmenuinnerinner');

    if (sidenavElm) {
      var linkElms = Util.findInnerElements(sidenavElm, 'a');

      var minHeight = 0;

      for (var i = 0; i < linkElms.length; i++) {
        var linkElm = linkElms[i];

        if (i === 0) { // assuming the first link will always be a one liner
          minHeight = linkElm.offsetHeight;
          continue;
        }

        var presentHeightClass = linkElm.getAttribute('data-height-class');

        if (linkElm.offsetHeight === minHeight) {
          if (presentHeightClass) {
            linkElm.classList.remove(presentHeightClass);
            linkElm.removeAttribute('data-height-class');
          }

          continue;
        }

        var relativeHeightRatio = Math.ceil(linkElm.offsetHeight / minHeight);

        // MyLearning._logDebug('_uiSidenavPatchLayout -> linkElm: ', {
        //   linkElm: linkElm,
        //   offsetHeight: linkElm.offsetHeight,
        //   minHeight: minHeight,
        //   relativeHeightRatio: relativeHeightRatio
        // });

        var heightClass = '-height-' + relativeHeightRatio;

        if (presentHeightClass) {
          if (presentHeightClass === heightClass) {
            continue;
          } else {
            linkElm.classList.remove(presentHeightClass);
          }
        }

        linkElm.classList.add(heightClass);
        linkElm.setAttribute('data-height-class', heightClass);
      }
    }
  }, timeoutDelay);
};

MyLearning._uiSidenavPatchClasses = function () {
  MyLearning._logDebug('_uiSidenavPatchClasses');

  var sidenavElm = document.getElementById('leftmenuinnerinner');

  MyLearning._logDebug('_uiSidenavPatchClasses -> sidenavElm: ', {
    sidenavElm: sidenavElm
  });

  if (sidenavElm) {
    var tutChapterElms = Util.findInnerElements(sidenavElm, '.tut_overview');

    for (var i = 0; i < tutChapterElms.length; i++) {
      var tutChapterElm = tutChapterElms[i];

      tutChapterElm.classList.add('overview_body');

      var tutChapterLinkElm = Util.findPreviousElementSibling(tutChapterElm);

      if (
        tutChapterLinkElm
        && tutChapterLinkElm.tagName === 'A'
      ) {
        tutChapterLinkElm.classList.add('overview_header');
      }
    }

    var refChapterElms = Util.findInnerElements(sidenavElm, '.ref_overview');

    for (var i = 0; i < refChapterElms.length; i++) {
      var refChapterElm = refChapterElms[i];

      refChapterElm.classList.add('overview_body');

      var refChapterLinkElm = Util.findPreviousElementSibling(refChapterElm);

      if (
        refChapterLinkElm
        && refChapterLinkElm.tagName === 'A'
      ) {
        refChapterLinkElm.classList.add('overview_header');
      }
    }

    // patch dropdown links
    var prevElm = null;

    for (var i = 0; i < sidenavElm.children.length; i++) {
      var childElm = sidenavElm.children[i];

      if (
        childElm.tagName === 'DIV'
        && childElm.classList.contains('overview_body')
        && prevElm !== null
        && prevElm.tagName === 'A'
      ) {
        var linkInnerIconElm = document.createElement('span');
        linkInnerIconElm.classList.add('w3s-accordion');

        prevElm.append(linkInnerIconElm);
      }

      prevElm = childElm;
    }
  }
};

MyLearning.uiSidenavPatch = function () {
  MyLearning._logDebug('uiSidenavPatch');

  MyLearning._uiSidenavPatchClasses();

  MyLearning._uiSidenavPatchLayout();

  window.addEventListener('resize', MyLearning._uiSidenavPatchLayout);
};

MyLearning._uiSidenavEnableCheckmarks = function () {
  MyLearning._logDebug('_uiSidenavEnableCheckmarks');

  var sidenavElm = document.getElementById('leftmenuinnerinner');

  if (sidenavElm) {
    sidenavElm.classList.add('checkmarks-enabled');
  }
};

MyLearning._uiSidenavFlagCompletedLinks = function (completed) {
  MyLearning._logDebug('_uiSidenavFlagCompletedLinks -> completed: ', completed);

  var sidenavElm = document.getElementById('leftmenuinnerinner');

  if (sidenavElm) {
    var completedMap = {};

    var currentHost = window.location.protocol + '//' + window.location.host;

    if (typeof completed !== 'undefined') {
      for (var i = 0; i < completed.length; i++) {
        completedMap[currentHost + '/' + completed[i]] = true;
      }
    }

    var linkElms = sidenavElm.getElementsByTagName('a');

    for (var i = 0; i < linkElms.length; i++) {
      var linkElm = linkElms[i];

      if (linkElm.classList.contains('overview_header')) {
        continue;
      }

      var linkHref = linkElm.href;

      // remove query string
      if (linkHref.indexOf('?') !== 0) {
        linkHref = linkHref.split('?')[0];
      }

      if (typeof completedMap[linkHref] !== 'undefined') {
        linkElm.classList.add('done');
      }
    }

    var linkChapterElms = Util.findInnerElements(sidenavElm, 'a.overview_header');

    if (linkChapterElms.length) {
      for (var i = 0; i < linkChapterElms.length; i++) {
        var linkChapterElm = linkChapterElms[i];

        // MyLearning._logDebug('_uiSidenavFlagCompletedLinks -> linkChapterElm: ', {
        //   linkChapterElm: linkChapterElm
        // });

        var chapterWrapperElm = Util.findNextElementSibling(linkChapterElm);

        // MyLearning._logDebug('_uiSidenavFlagCompletedLinks -> chapterWrapperElm: ', {
        //   chapterWrapperElm: chapterWrapperElm,
        // });

        if (
          chapterWrapperElm
          && chapterWrapperElm.classList.contains('overview_body')
        ) {

          var chapterLinkElms = Util.findInnerElements(chapterWrapperElm, 'a');

          var chapterCompleted = true;

          for (var j = 0; j < chapterLinkElms.length; j++) {
            var chapterLinkElm = chapterLinkElms[j];

            if (!chapterLinkElm.classList.contains('done')) {
              chapterCompleted = false;
              break;
            }
          }

          if (chapterCompleted) {
            linkChapterElm.classList.add('done');
          }
        }
      }
    }
  }
};

MyLearning._pushAnonymouslyTrackedProgress = function (progressTrackingMeta, callback) {
  var output = {
    error: {
      code: '1',
      description: 'Failed performing "MyLearning._pushAnonymouslyTrackedProgress"'
    },
    data: null,
  };

  Util.fetch({
    context: 'Pathfinder',
    method: 'POST',
    url: MyLearning._apiBaseUrl + '/api/classic/bulk-upsert-progress',
    headers: {
      'Content-Type': 'application/json'
    },
    data: JSON.stringify({
      currentUts: Util.getCurrentUts(),
      createdUts: progressTrackingMeta.createdUts,
      updatedUts: progressTrackingMeta.updatedUts,
      size: progressTrackingMeta.size,
      list: progressTrackingMeta.list,
      sig: progressTrackingMeta.sig,
      currentTopicId: MyLearning.topicId,
    }),
    sendCookies: false,
    withUserSession: true,
    prepareResponseData: true,
    callback: function (fetchRes) {
      MyLearning._logDebug('_pushAnonymouslyTrackedProgress -> fetchRes: ', fetchRes);

      if (fetchRes.error.code === '0') {
        output.data = fetchRes.data;
      } else {
        output.error = fetchRes.error;
      }

      if (output.error.code === '1') {
        output.error = { code: '0' };
      }

      if (typeof callback !== 'undefined') {
        callback(output);
      }
    }
  });
};

MyLearning._fetchAnonymousProgressTrackingMeta = function () {
  MyLearning._logDebug('_fetchAnonymousProgressTrackingMeta');

  var output = {
    error: {
      code: '1',
      description: 'Failed performing "MyLearning._fetchAnonymousProgressTrackingMeta"'
    },
    data: null,
  };

  try {
    var progressTrackingMetaJsonStr = localStorage.getItem('MyLearning.tracking');

    if (progressTrackingMetaJsonStr) {
      var progressTrackingMetaParseRes = Util.parseJson(progressTrackingMetaJsonStr, [
        'version',
        'createdUts',
        'updatedUts',
        'size',
        'list',
        'sig'
      ]);

      if (progressTrackingMetaParseRes.error.code === '0') {
        var progressTrackingMetaRaw = progressTrackingMetaParseRes.data;

        if (progressTrackingMetaRaw.version !== MyLearning.progressTrackingVersion) {
          output.error = {
            code: 'VERSION_MISMATCH',
            description: 'Cached progress version mismatch',
            meta: progressTrackingMetaRaw
          };

          localStorage.removeItem('MyLearning.tracking');

          return output;
        }

        output.data = {
          raw: progressTrackingMetaJsonStr,
          prepared: progressTrackingMetaRaw
        };

        if (Array.isArray(progressTrackingMetaRaw.list)) {
          if (!progressTrackingMetaRaw.list.length) {
            output.error = {
              code: 'NOT_FOUND',
              description: 'No tracking data found'
            };

            localStorage.removeItem('MyLearning.tracking');
          }
        } else {
          output.error = {
            code: 'INVALID_DATA_TYPE',
            description: 'Cached progress is corrupted',
            meta: progressTrackingMetaRaw
          };

          localStorage.removeItem('MyLearning.tracking');
        }
      } else {
        output.error = {
          code: 'PARSE_ERROR',
          description: 'Failed parsing progressTrackingMetaRaw',
          meta: progressTrackingMetaParseRes.error
        };

        localStorage.removeItem('MyLearning.tracking');
      }
    } else {
      output.error = {
        code: 'NOT_FOUND',
        description: 'No tracking data found'
      };
    }

    if (output.error.code === '1') {
      output.error = { code: '0' };
    }
  } catch (exc) {
    MyLearning._logError('MyLearning._fetchAnonymousProgressTrackingMeta', exc);
    output.error = Util.getMetaPreparedFromException(exc);
    output.error.code = '2';
  }

  return output;
};

MyLearning._handleAnonymousProgressTracking = function () {
  MyLearning._logDebug('_handleAnonymousProgressTracking');
  return;

  var fetchAnonymousProgressTrackingMetaRes = MyLearning._fetchAnonymousProgressTrackingMeta();
  MyLearning._logDebug('fetchAnonymousProgressTrackingMetaRes: ', fetchAnonymousProgressTrackingMetaRes);

  // storing the anonymous progress for later
  MyLearning._fetchAnonymousProgressTrackingMetaRes = fetchAnonymousProgressTrackingMetaRes;

  // anonymous progress is present
  if (fetchAnonymousProgressTrackingMetaRes.error.code === '0') {
    var anonymousProgressTrackingMeta = fetchAnonymousProgressTrackingMetaRes.data;

    MyLearning._uiSidenavFlagCompletedLinks(MyLearning._getPathsListFromAnonymousProgressTrackingList(anonymousProgressTrackingMeta.prepared.list));

    if (UserSession.loggedIn === true) {
      MyLearning._pushAnonymouslyTrackedProgress(anonymousProgressTrackingMeta.prepared, function (res) {
        MyLearning._logDebug('_pushAnonymouslyTrackedProgress -> res: ', res);

        if (res.error.code === '0') {
          localStorage.removeItem('MyLearning.tracking');

          if (typeof res.data.completed !== 'undefined') {
            MyLearning._uiSidenavFlagCompletedLinks(res.data.completed);
          }
        } else {
          MyLearning._logError('_pushAnonymouslyTrackedProgress -> res: ', res);
        }
      });
    }
  }

  if (UserSession.loggedIn === false) {
    window.addEventListener('scroll', MyLearning._checkIfGotToTheBottomOfThePage);

    MyLearning._checkIfGotToTheBottomOfThePage();
  }
};

MyLearning._pathIsNotInAnonymousProgressTrackingList = function (list, path) {
  MyLearning._logDebug('_pathIsNotInAnonymousProgressTrackingList -> list, path: ', {
    list: list,
    path: path
  });

  if (!list.length) {
    return true;
  }

  var isPresent = false;

  for (var i = 0; i < list.length; i++) {
    if (list[i][0] === path) {
      isPresent = true;
      break;
    }
  }

  return !isPresent;
};

MyLearning._getPathsListFromAnonymousProgressTrackingList = function (list) {
  var output = [];

  for (var i = 0; i < list.length; i++) {
    output.push(list[i][0]);
  }

  return output;
};

MyLearning._handleFinishedPageAnonymous = function () {
  MyLearning._logDebug('_handleFinishedPageAnonymous');
  return;

  var fetchAnonymousProgressTrackingMetaRes = MyLearning._fetchAnonymousProgressTrackingMetaRes;

  var anonymousProgressTrackingMeta;

  var currentUts = Util.getCurrentUts();

  // anonymous progress is present
  if (fetchAnonymousProgressTrackingMetaRes.error.code === '0') {
    anonymousProgressTrackingMeta = fetchAnonymousProgressTrackingMetaRes.data.prepared;
  } else {
    anonymousProgressTrackingMeta = {
      version: MyLearning.progressTrackingVersion,
      createdUts: currentUts,
      updatedUts: currentUts,
      size: 0,
      list: [],
      sig: '',
    };
  }

  var currentPath = Util.removeStrPrefix(window.location.pathname, '/');

  MyLearning._logDebug('_handleFinishedPageAnonymous -> anonymousProgressTrackingMeta: ', anonymousProgressTrackingMeta);

  var saveProgressInfoModalNotificationRate = 5; // every 5th lesson

  var nrOfVisitedPages = anonymousProgressTrackingMeta.list.length;

  if (anonymousProgressTrackingMeta.list.length < MyLearning.localStorageTrackingMaxCount) {
    if (MyLearning._pathIsNotInAnonymousProgressTrackingList(anonymousProgressTrackingMeta.list, currentPath)) {
      Util.attachScript({
        id: 'my-learning-cbups-script',
        path: '/lib/my-learning/cbups.js',
        entityName: 'Cbups',
        callback: function (res) {
          if (res.error.code === '0') {
            var listItem = [currentPath, Cbups.generateId(currentPath)];

            anonymousProgressTrackingMeta.size += Util.getStrByteSize(JSON.stringify(listItem));

            var listItemIndex = anonymousProgressTrackingMeta.list.length;

            anonymousProgressTrackingMeta.list.push(listItem);

            anonymousProgressTrackingMeta.sig = Cbups.getSignature({
              prevSig: anonymousProgressTrackingMeta.sig,
              list: anonymousProgressTrackingMeta.list,
              index: listItemIndex
            });

            anonymousProgressTrackingMeta.updatedUts = Util.getCurrentUts();

            localStorage.setItem('MyLearning.tracking', JSON.stringify(anonymousProgressTrackingMeta));

            MyLearning._uiSidenavFlagCompletedLinks([currentPath]);

            MyLearning._logDebug('_handleFinishedPageAnonymous -> anonymousProgressTrackingMeta -> updated: ', anonymousProgressTrackingMeta);

            if (MyLearningSaveProgressInfoModal._doNotShowAgain()) {
              return;
            }

            nrOfVisitedPages = anonymousProgressTrackingMeta.list.length;

            if ((anonymousProgressTrackingMeta.list.length % saveProgressInfoModalNotificationRate) === 0) {
              MyLearningSaveProgressInfoModal._show({
                nrOfVisitedPages: nrOfVisitedPages,
                nrOfInitialRenders: Math.floor(nrOfVisitedPages / saveProgressInfoModalNotificationRate)
              });
            } else if (MyLearningSaveProgressInfoModal._renderedButNotSeen()) {
              MyLearningSaveProgressInfoModal._show({
                nrOfVisitedPages: nrOfVisitedPages,
                nrOfInitialRenders: Math.floor(nrOfVisitedPages / saveProgressInfoModalNotificationRate)
              });
            }
          } else {
            console.error('Failed loading my learning cbups -> error: ', res.error);
          }
        }
      });
    } else {
      if (MyLearningSaveProgressInfoModal._renderedButNotSeen()) {
        MyLearningSaveProgressInfoModal._show({
          nrOfVisitedPages: nrOfVisitedPages,
          nrOfInitialRenders: Math.floor(nrOfVisitedPages / saveProgressInfoModalNotificationRate)
        });
      }
    }
  } else {
    MyLearning._logError('_handleFinishedPageAnonymous -> user reached max tracking limit');

    if (MyLearningSaveProgressInfoModal._renderedButNotSeen()) {
      MyLearningSaveProgressInfoModal._show({
        nrOfVisitedPages: nrOfVisitedPages,
        nrOfInitialRenders: Math.floor(nrOfVisitedPages / saveProgressInfoModalNotificationRate)
      });
    }
  }
};

MyLearning.init = function () {
  MyLearning._logDebug('init');

  Util.objFieldOnSetCallback({
    scopeRef: MyLearning,
    fieldName: 'lessonMeta', // set by MyLearning only after it's fetched from the server
    callback: function (res) {
      MyLearning._logDebug('MyLearning.lessonMeta -> res: ', res);

      if (res.error.code === '0') {
        MyLearning._uiSidenavFlagCompletedLinks(res.data.completed);
      } else {
        MyLearning._logError('MyLearning.lessonMeta -> res: ', res);
      }
    }
  });

  Util.objFieldOnSetCallback({
    scopeRef: UserSession,
    fieldName: 'loggedIn', // not null after UserSession.processUserSession is done processing
    fieldValue: null,
    fieldValueEquals: false,
    callback: function (res) {
      MyLearning._logDebug('UserSession.loggedIn -> res: ', res);

      if (res.error.code === '0') {
        // MyLearning._handleAnonymousProgressTracking();

        if (UserSession.loggedIn === true) {
          MyLearning._uiSidenavEnableCheckmarks();
        }
      } else {
        MyLearning._logError('UserSession.loggedIn -> res: ', res);
      }
    }
  });

  if (MyLearning.isIframed()) {
    MyLearning._iframeInit();
  }

  // MyLearningSaveProgressInfoModal._show({
  //   nrOfVisitedPages: 5,
  //   nrOfInitialRenders: 1
  // });
};

MyLearningSaveProgressInfoModal = {
  _elm: null,
  _rendered: false,
  _seen: false,
};

MyLearningSaveProgressInfoModal._onResize = function () {
  if (!MyLearningSaveProgressInfoModal._rendered) {
    return;
  }

  var modalElm = MyLearningSaveProgressInfoModal._elm;

  var sideNavCalcElm = document.getElementById('w3s-sidenav-width-calc');

  if (sideNavCalcElm) {
    var sideNavWidth = Util.getElementWidth(sideNavCalcElm);

    modalElm.style.width = sideNavWidth ? sideNavWidth : '205px';
  }
};

MyLearningSaveProgressInfoModal._onClose = function (event) {
  event.stopPropagation();

  if (!MyLearningSaveProgressInfoModal._rendered) {
    return;
  }

  MyLearningSaveProgressInfoModal._detachEvents();

  MyLearningSaveProgressInfoModal._rendered = false;

  setTimeout(function () {
    MyLearningSaveProgressInfoModal._elm.remove();

    MyLearningSaveProgressInfoModal._elm = null;
  }, 300);
};

MyLearningSaveProgressInfoModal._onDontShowAgainToggle = function () {
  if (!MyLearningSaveProgressInfoModal._rendered) {
    return;
  }

  var modalElm = MyLearningSaveProgressInfoModal._elm;

  var checkboxWrapperElm = Util.getInnerElement(modalElm, '.-dont-show-again');

  if (checkboxWrapperElm) {
    if (checkboxWrapperElm.classList.contains('-checked')) {
      checkboxWrapperElm.classList.remove('-checked');

      if (Cookies.get('MyLearning.saveProgressInfoModalState') === '-1') {
        Cookies.remove('MyLearning.saveProgressInfoModalState');
      }
    } else {
      checkboxWrapperElm.classList.add('-checked');

      Cookies.set('MyLearning.saveProgressInfoModalState', '-1', { expires: 365 }); // do not show again during one year
    }
  }
};

MyLearningSaveProgressInfoModal._sideNavIsVisible = function () {
  var sideNavIsVisible = Util.getWindowWidth() > 992;

  MyLearning._logDebug('MyLearningSaveProgressInfoModal._sideNavIsVisible: ', sideNavIsVisible);

  return sideNavIsVisible;
};

MyLearningSaveProgressInfoModal._attachEvents = function () {
  if (!MyLearningSaveProgressInfoModal._rendered) {
    return;
  }

  var modalElm = MyLearningSaveProgressInfoModal._elm;

  window.addEventListener('resize', MyLearningSaveProgressInfoModal._onResize);

  Util.getInnerElement(modalElm, '.-close-btn').addEventListener('click', MyLearningSaveProgressInfoModal._onClose);

  Util.getInnerElement(modalElm, '.-dont-show-again').addEventListener('click', MyLearningSaveProgressInfoModal._onDontShowAgainToggle);

  document.getElementById('sidenav').addEventListener('w3sSideNavOpen', MyLearningSaveProgressInfoModal._onSideNavOpen);

  setTimeout(function () {
    if (MyLearningSaveProgressInfoModal._sideNavIsVisible()) {
      MyLearningSaveProgressInfoModal._seen = true;

      Cookies.set('MyLearning.saveProgressInfoModalState', '2'); // rendered and seen
    }
  }, 300); // slide up animation duration
};

MyLearningSaveProgressInfoModal._onSideNavOpen = function (event) {
  MyLearning._logDebug('MyLearningSaveProgressInfoModal._onSideNavOpen -> event: ', event);

  if (
    MyLearningSaveProgressInfoModal._rendered
    && !MyLearningSaveProgressInfoModal._seen
  ) {
    setTimeout(function () {
      MyLearningSaveProgressInfoModal._seen = true;

      Cookies.set('MyLearning.saveProgressInfoModalState', '2'); // rendered and seen
    }, 300); // slide up animation duration
  }
};

MyLearningSaveProgressInfoModal._detachEvents = function () {
  if (!MyLearningSaveProgressInfoModal._rendered) {
    return;
  }

  var modalElm = MyLearningSaveProgressInfoModal._elm;

  window.removeEventListener('resize', MyLearningSaveProgressInfoModal._onResize);

  Util.getInnerElement(modalElm, '.-close-btn').removeEventListener('click', MyLearningSaveProgressInfoModal._onClose);

  Util.getInnerElement(modalElm, '.-dont-show-again').removeEventListener('click', MyLearningSaveProgressInfoModal._onDontShowAgainToggle);

  document.getElementById('sidenav').removeEventListener('w3sSideNavOpen', MyLearningSaveProgressInfoModal._onSideNavOpen);
};

MyLearningSaveProgressInfoModal._renderedButNotSeen = function () {
  return Cookies.get('MyLearning.saveProgressInfoModalState') === '1';
};

MyLearningSaveProgressInfoModal._doNotShowAgain = function () {
  return Cookies.get('MyLearning.saveProgressInfoModalState') === '-1';
};

MyLearningSaveProgressInfoModal._show = function (prs) {
  if (UserSession.loggedIn === true) {
    return;
  }

  var nrOfVisitedPages = prs.nrOfVisitedPages;
  var nrOfInitialRenders = prs.nrOfInitialRenders; // Math.floor(nrOfVisitedPages / saveProgressInfoModalNotificationRate)

  MyLearning._logDebug('MyLearningSaveProgressInfoModal._show -> prs: ', prs);

  // if on desktop after 50 visited pages
  if (
    MyLearningSaveProgressInfoModal._sideNavIsVisible()
    && nrOfInitialRenders > 10
  ) {
    return;
  }

  MyLearning.uiFetchTemplate('/lib/my-learning/html/user-anonymous/save-progress-info-modal.tmpl.html', function (templateFetchRes) {
    if (templateFetchRes.error.code === '0') {
      if (MyLearningSaveProgressInfoModal._rendered) {
        return;
      }

      MyLearningSaveProgressInfoModal._rendered = true;

      var wrapperElm = document.getElementById('leftmenuinner');
      var wrapperInnerElm = document.getElementById('leftmenuinnerinner');

      if (!wrapperElm || !wrapperInnerElm) {
        MyLearning._logError('MyLearningSaveProgressInfoModal._show -> wrapper elm not found');
        return;
      }

      var sideNavCalcElm = document.createElement('div');
      sideNavCalcElm.id = 'w3s-sidenav-width-calc';
      sideNavCalcElm.className = '-width-calc';

      wrapperInnerElm.appendChild(sideNavCalcElm);

      var sideNavWidth = Util.getElementWidth(sideNavCalcElm);

      var tmpPlaceholderElm = document.createElement('div');
      tmpPlaceholderElm.innerHTML = MyLearning.uiPrepareTemplateHtml(templateFetchRes.data, {
        width: sideNavWidth ? sideNavWidth : '205px',
        nrOfVisitedPages: nrOfVisitedPages,
        // doNotShowAgain: nrOfInitialRenders > 10 ? 'checked' : 'not-checked'
        doNotShowAgain: 'not-checked',
        authRedirectUrl: encodeURIComponent(window.location.href)
      });

      wrapperElm.appendChild(tmpPlaceholderElm.firstChild);

      var modalElm = document.getElementById('w3s-myl-save-progress-info-modal');

      if (!modalElm) {
        MyLearning._logError('MyLearningSaveProgressInfoModal._show -> modal elm not found');
        return;
      }

      MyLearningSaveProgressInfoModal._elm = modalElm;

      Cookies.set('MyLearning.saveProgressInfoModalState', '1'); // rendered but not seen

      MyLearningSaveProgressInfoModal._attachEvents();
    }
  });
};

MyLearning.init();

// if (window.addEventListener) {
//   window.addEventListener('load', MyLearning.init);
// } else if (window.attachEvent) {
//   window.attachEvent('onload', MyLearning.init);
// }
