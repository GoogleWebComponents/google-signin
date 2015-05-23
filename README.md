google-signin
================

See the [component page](https://googlewebcomponents.github.io/google-signin) for more information.

Data flow:

# google-signin

      on-iron-signal-google-auth-scopes="authScopesChange"

globals:

authorizedScopes
extraScopes
defaultHandler

mergeExtraScopes(authScopes)
  returns authScopes + unauthorized extraScopes

properties
    in: appPackageName
    in: clientId
    in: cookiePolicy
    in: requestVisibleActions
    in: scopes

    out: signedIn
    out: additonalAuth
  ui
    brand
    height
    fill
    labelAdditional
    labelSignin
    labelSignout
    raised
    theme
    width

events:

  google-auth-request:
    on-iron-signal-google-auth-request="authRequestScopes"
    data: google-signin-aware scope array
    signaled from google-signin-aware
    processed in authRequestScopes()
      adds scopes to extra-scopes


  google-auth-user:
    on-iron-signal-google-auth-user="authUserChange"
    emitted from auth.currentUser.listen callback
    data: user: newUser
    processed in google-signin.authUserChange

    processed in google-signin-aware.authUserChange

x  google-signin-necessary:
    emitted from checkScopes() if there are non-granted scopes

x  google-signin-success
    emitted from authUserChange if signed in

x  google-signed-out
    emitted from authUserChange if signed out

x  google-signout-attempted
    emitted before signout starts

x  google-signin-aware-success
    emitted when signin-aware is logged in, and all auth granted

x  google-signin-aware-signed-out
    emitted when signin-aware user logs out

    note: no event if user signed in, but not authorized

  gapi:
    loadAuth: when js-api loads

    gapi.load

      gapi.init(clientId, cookiePolicy, scopes)

      auth.currentUser.listen

      gapi.auth2.getAuthInstance().signIn(params)
        params are appPackageName, clientId, cookiePolicy, requestVisibleActions, and all scopes requested so far

      gapi.auth2.getAuthInstance().signOut()

google-signin
  - manages display
  - passes arguments to google-signin-aware
  - gets signedIn, authorized from signin-aware

google-signin-aware
  - passes auth-props to auth-engine
  - signIn, signOut to auth-engine

  - receives currentUser, currentUserUpdated (if user did not change, but auths did)
  - registers with auth-engine
  - hasPlusScopes

auth-engine
  - loadLibrary, init
  - auth.currentUser.listen
  - areAuthorizationsGranted(authList)
  - areAllAuthorizationsGranted(
  https://developers.google.com/youtube/v3/guides/authentication
  - signIn (approval_prompt parameter)
  - keeps list of google-signin-aware

Rewrite:
  can ask for permissions without button, just <google-signin-aware>
  keeps track of all


GoogelAuth.signIn(params) gets IMMEDIATE_FAILED first time, second time it works

I am using Google Sign-In Javascript Client (gapi.oauth2). I've refactored some code, and now I am stumped. This code:


    params = { scope: "profile https://www.googleapis.com/auth/calendar",
              clientid: "XXX",
              cookiepolicy: "single_host_origin" }

    gapi.auth2.getAuthInstance().signIn(params).then(
          function success(newUser) {},
          function error(error) { console.error(error); });

Fails with: `{type: "tokenFailed", idpId: "google", error: "IMMEDIATE_FAILED"}`
