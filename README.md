[npm:img]: https://img.shields.io/npm/v/react-social-login.svg?style=flat-square
[npm:url]: https://www.npmjs.org/package/react-social-login

[standard:img]: https://img.shields.io/badge/code%20style-standard-brightgreen.svg?style=flat-square
[standard:url]: http://standardjs.com

[dependencies:img]: https://david-dm.org/deepakaggarwal7/react-social-login.svg?style=flat-square
[dependencies:url]: https://github.com/deepakaggarwal7/react-social-login/blob/master/package.json#L33

[license:img]: https://img.shields.io/badge/License-MIT-brightgreen.svg?style=flat-square
[license:url]: https://opensource.org/licenses/MIT

# React Social Login &middot; [![NPM version][npm:img]][npm:url] &middot; [![Standard - JavaScript Style Guide][standard:img]][standard:url] &middot; [![Dependencies][dependencies:img]][dependencies:url] &middot; [![License: MIT][license:img]][license:url]

React Social Login is an HOC which provides social login through multiple providers.

**Currently supports Amazon, Facebook, GitHub, Google, Instagram and LinkedIn as providers (more to come!)**

## Motivation

 * Having a component that doesn't dictates the HTML
 * All-in-One login component for different social providers
 * Takes care of warnings from provider's SDKs when multiple instances are placed
 * Kind of re-birth of my previous .Net driven similar open source - SocialAuth.NET

## Demo

Edit `appId` props with your own ones in `demo/index.js` file and build demo:

```shell
$ npm start
```

You can then view the demo at [https://localhost:8080][demo].

For GitHub provider, see [GitHub specifics][githubspecifics] first.

## Install

```shell
$ npm install --save react-social-login
```

## Usage

Create the component of your choice and transform it into a SocialLogin component.

**SocialButton.js**

```js
import React from 'react'
import SocialLogin from 'react-social-login'

const Button = ({ children, triggerLogin, ...props }) => (
  <button onClick={triggerLogin} {...props}>
    { children }
  </div>
)

export default SocialLogin(Button)
```

Then, use it like a normal component.

**index.js**

```js
import React from 'react'
import ReactDOM from 'react-dom'

import SocialButton from './SocialButton'

const handleSocialLogin = (user) => {
  console.log(user)
}

const handleSocialLoginFailure = (err) => {
  console.error(err)
}

ReactDOM.render(
  <div>
    <SocialButton
      provider='facebook'
      appId='YOUR_APP_ID'
      onLoginSuccess={handleSocialLogin}
      onLoginFailure={handleSocialLoginFailure}
    >
      Login with Facebook
    </SocialButton>
  </div>,
  document.getElementById('app')
)
```

### Reference

Raw component props (before transform):

| Prop  | Default  | Type / Values  | Description  |
|---|---|---|---|
| appId  | —  | string  | Your app identifier (see [find my appId][findmyappid])  |
| autoCleanUri  | false  | boolean  | Enable auto URI cleaning with OAuth callbacks  |
| autoLogin  | false  | boolean  | Enable auto login on `componentDidMount`  |
| gatekeeper  | —  | string  | Gatekeeper URL to use for GitHub OAuth support (see [GitHub specifics][githubspecifics])  |
| onLoginFailure  | —  | function  | Callback on login fail  |
| onLoginSuccess  | —  | function  | Callback on login success  |
| provider  | —  | `amazon`, `facebook`, `github`, `google`, `instagram`, `linkedin`  | Social provider to use  |
| redirect  | -  | string  | URL to redirect after login (available for `github` and `instagram` only)  |
| scope  | -  | array, string  | An array or string of scopes to be forwarded in request. Current support is for [google scopes][googlescopes] only (more to come)  |
| any other prop  | —  | —  | Any other prop will be forwarded to your component  |

*Note about `redirect`: if you are redirecting on root (eg: https://localhost:8080), you **have** to omit the trailing slash.*

Transformed component props:

| Prop  | Type  | Description  |
|---|---|---|
| triggerLogin  | function  | Function to trigger login process, usually attached to an event listener  |
| all your props  | —  | All props from your original component, minus SocialLogin specific props  |

## Old component support

We decided to keep the old behavior as a fallback, it only supports `facebook`, `google` and `linkedin` providers and is available as a named export:

```js
import React from 'react'
import ReactDOM from 'react-dom'
import { OldSocialLogin as SocialLogin } from 'react-social-login'

const handleSocialLogin = (user, err) => {
  console.log(user)
  console.log(err)
}

ReactDOM.render(
  <div>
    <SocialLogin
      provider='facebook'
      appId='YOUR_APP_ID'
      callback={handleSocialLogin}
    >
      <button>Login with Google</button>
    </SocialLogin>
  </div>,
  document.getElementById('app')
)
```

## Build

Though not mandatory, it is recommended to use latest npm5 to match lockfile versions.

```shell
$ npm install
$ npm run build
```

## Miscellaneous

### Find my appId

#### Amazon

See [Amazon developers documentation][amazondoc].

#### Facebook

See [facebook for developers documentation][fb4devdoc].

#### GitHub (see [GitHub specifics][githubspecifics])

* Basic authentication method using personal access tokens: see [GitHub Help][githubhelp].
* OAuth authentication: see [GitHub Developer guide][githubdoc].

#### Google

See [Google Sign-In for Websites guide][gsignindoc].

#### Instagram

See [Instagram developers documentation][instadoc].

#### LinkedIn

See `Where can I find my API key?` section on the [FAQ][linkedinfaq].

### GitHub specifics

GitHub provider is implemented in two different modes:

* One using [GitHub Personal Tokens][ghpersonaltokens]
* Another using GitHub OAuth

#### GitHub Personal Tokens mode

Actually, this one is more a hacky way to get user profile than a way to really *connect* your app like OAuth does.

Plus, it requires from users to create their personal token from their GitHub account, which is not a good experience for them.

This mode is the default if you do not provide `gatekeeper` prop and will try to use the `appId` prop to get user data. Anyway, we **strongly advise you to use the GitHub OAuth authentication flow**.

#### GitHub OAuth

If you provide a `gatekeeper` prop, this mode will be active and will use a server of your own to fetch GitHub OAuth access token. This is a [know issue of GitHub][ghoauthwebflowissue].

The simplest way to setup this mode is to use the [Gatekeeper project][gatekeeper]. Just follow setup instructions then tell RSL to use it:

```
<SocialLogin
  provider='github'
  gatekeeper='http://localhost:9999'
  appId='YOUR_GITHUB_CLIENT_ID'
  redirect='http://localhost:8080'
>
  Login with GitHub OAuth
</SocialLogin>
```

You can also implement it your own way but you must use the same routing than `Gatekeeper` (`/authenticate/:code`) and return a JSON response containing a `token` or `error` property (it will also throw if it doesn't find `token`).

#### Special instructions to run demo

RSL demo is served over https with `webpack-dev-server`. This is a requirement of Amazon Login SDK. `Gatekeeper` is served over insecure http so you will have to serve the demo through http also to work with GitHub (but it will break Amazon):

```shell
$ npm run start:insecure
```

## Change Log

__v2.0.0__ [26 Feb 2017]
* Use small case for providers
* Linkedin support added along with previous google and facebook
* A lot of refactoring done
* Uses Webpack 2.x
__Huge  Thanks to  Nicolas Goudry for his generous contribution __

__v2.0.1__ [24 June 2017] merged pull request [#15](https://github.com/deepakaggarwal7/react-social-login/pull/15)
 * Facebook error
 * code styling
 * unnecessary console logs
 * pre-commit lint

__v3.0.0__ [30 July 2017] merged pull request [#19](https://github.com/deepakaggarwal7/react-social-login/pull/19)
 * Rewrote as HOC
 * Instagram
 * GitHub
 * Auto login
 * Better error handling
 * Various fixes

__v3.1.0__ [01 August 2017] merged pull request [#20](https://github.com/deepakaggarwal7/react-social-login/pull/20)
 * Amazon
 * https

__v3.2.0__ [14 September 2017] 3.2.0
 * Custom Google scopes
 * Multiple Google buttons
 * GitHub doc

__v3.2.1__ [06 October 2017] React 16, better build, update dep, additions and fixes
 * Update React to v16
 * Update all dependencies to latest stable versions
 * Optimize webpack build
 * Return full googleAuthResponse
 * Preserve redirect url query string and hash
 * Cancel loading on componentWillUnmount
 * Various fixes

## Tests

TBD

## Main contributors
* [Deepak Aggarwal][ghdeepak]
* [Nicolas Goudry][ghngoudry]

## Kind contributors
* Kamran Ahmed : custom Google scopes
* Adrien Cohen : multiple Google buttons
* Jason Loo : full google auth response


[demo]: https://localhost:8080
[findmyappid]: #find-my-appid
[fb4devdoc]: https://developers.facebook.com/docs/apps/register
[githubhelp]: https://help.github.com/articles/creating-a-personal-access-token-for-the-command-line
[githubdoc]: https://developer.github.com/apps/building-integrations/setting-up-and-registering-oauth-apps/registering-oauth-apps
[gsignindoc]: https://developers.google.com/identity/sign-in/web/devconsole-project
[instadoc]: https://www.instagram.com/developer
[linkedinfaq]: https://developer.linkedin.com/support/faq
[ghpersonaltokens]: https://github.com/blog/1509-personal-api-tokens
[ghoauthwebflowissue]: https://github.com/isaacs/github/issues/330
[gatekeeper]: https://github.com/prose/gatekeeper
[githubspecifics]: #github-specifics
[amazondoc]: https://developer.amazon.com/public/apis/engage/login-with-amazon/docs/register_web.html
[googlescopes]: https://developers.google.com/identity/protocols/googlescopes
[ghngoudry]: https://github.com/nicolas-goudry
[ghdeepak]: https://github.com/deepakaggarwal7
