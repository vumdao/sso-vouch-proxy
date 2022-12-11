<p align="center">
  <a href="https://dev.to/vumdao">
    <img alt="Single-Sign-On By Vouch Proxy And AWS Cognito" src="images/cover.png" width="700" />
  </a>
</p>
<h1 align="center">
  <div><b>Single-Sign-On By Vouch Proxy And AWS Cognito</b></div>
</h1>

## Abstract
- This post introduces vouch proxy as an solution to provide single-sign on which supports many OAuth and OIDC login providers and can enforce authentication to here is AWS cognito user pool.

## Table Of Contents
* [Vouch Proxy Overview](#Vouch-Proxy-Overview)
* [What Exactly Vouch Proxy Does?](#What-Exactly-Vouch-Proxy-Does?)
* [How does vouch-proxy protect the domains?](#How-does-vouch-proxy-protect-the-domains?)
* [Setup AWS Cognito as authorize provider](#Setup-AWS-Cognito-as-authorize-provider)
* [Troubleshooting](#Troubleshooting)
* [Conclusion](#Conclusion)

---

## 🚀 **Vouch Proxy Overvieww** <a name="Vouch-Proxy-Overview"></a>
- [Vouch Proxy](https://github.com/vouch/vouch-proxy) - An SSO solution for Nginx using the auth_request module. Vouch Proxy can protect all of your websites at once.
- Vouch Proxy supports many OAuth and OIDC login providers and can enforce authentication to... This blog introduces applying vouch-proxy with AWS Cognito

## 🚀 **What Exactly Vouch Proxy Does** <a name="What-Exactly-Vouch-Proxy-Does"></a>
- **Diagram**

  <img src=images/vp-diagram.png withd=1100>

- **The flows**
  1. User access Solr Admin UI through https://sso.simflexcloud.com/solr
  2. The request is forwarded to ALB -> NGINX
  3. Nginx authorizes the request by calling/validating request using vouch-proxy through the internal endpoint `http://vouch:9090/validate`
  4. Vouch-proxy validates and returns 200 if user already has the cache login token or returns `401` if not yet perform login
     1. With 200 code, the request is redirected to target location here is Solr UI admin
     2. With 401 code, the request is redirected to `https://vouch.simflexcloud.com/login?url=https://$http_host$request_uri&vouch-failcount=$auth_resp_failcount&X-Vouch-Token=$auth_resp_jwt&error=$auth_resp_err>` to login, if the login is successful, the request is redirected to target location here is Solr UI admin
  5. Default redirect is the main page or marketing page or anything

## 🚀 **How does vouch-proxy protect the domains?** <a name="How-does-vouch-proxy-protect-the-domains?"></a>
- Vouch-proxy (VP) supports many OAuth and OIDC login providers to authentication
- Futhermore, vouch-proxy whitelists the domains, not only the access domain but also the domain of email login. The cookie domain, the domain that VP is served from and the protected app must all be the same domain. For example, if we just whitelist the app domain such simflexcloud.com but the email is vu.dao@gmail.com it will not authorize the user
  ```
  2022-07-07T15:58:12.716Z        WARN    /auth User is not authorized: verifyUser: Email vu.dao@gmail.com is not within a Vouch Proxy managed domain . Please try again or seek support from your administrator
  ```

## 🚀 **Setup AWS Cognito as authorize provider** <a name="Setup-AWS-Cognito-as-authorize-provider"></a>
- Cognito userpool application client

  <img src=images/cognito-sso.png width=1100>

- Setup vouch-proxy config with AWS cognito, replace the URLs with ones setup from cognito
  ```
  vouch:
    logLevel: debug
    port: 9090

    cookie:
      secure: false

    domains: # app domain and mail domain only
    - simflexcloud.com
    - gmail.com

  oauth:
    provider: oidc
    client_id: AWS_COGNITO_VOUCH_CLIENT_ID
    auth_url: <https://{yourCognitoDomain}.amazoncognito.com/oauth2/authorize>
    token_url: <https://{yourCognitoDomain}.amazoncognito.com/oauth2/token>
    user_info_url: <https://{yourCognitoDomain}.amazoncognito.com/oauth2/userInfo>
    scopes:
    - openid
    - email
    - profile
    callback_url: <https://{callbackURL}/auth>
  ```

## 🚀 **Troubleshooting** <a name="Troubleshooting"></a>
- Getting started to align between Nginx, Vouch Proxy and your identity provider (IdP) can be tricky, vouch-proxy provides config to enable testing mode
  ```
  vouch:
    testing: true
    logLevel: debug
  ```

- With testing enable, the UI will step us to each phase of redirecting

  <img src=images/vp-1.png width=1100>

  <img src=images/vp-2.png width=1100>

## 🚀 **Conclusion** <a name="Conclusion"></a>
- Follow the nginx config in [nginx]() and Dockerfile to build the `nginx-gw`
- Use the `quay.io/vouch/vouch-proxy` image to start vouch-proxy
---

<h3 align="center">
  <a href="https://dev.to/vumdao">:stars: Blog</a>
  <span> · </span>
  <a href="https://github.com/vumdao/sso-vouch-proxy">Github</a>
  <span> · </span>
  <a href="https://stackoverflow.com/users/11430272/vumdao">stackoverflow</a>
  <span> · </span>
  <a href="https://www.linkedin.com/in/vu-dao-9280ab43/">Linkedin</a>
  <span> · </span>
  <a href="https://www.linkedin.com/groups/12488649/">Group</a>
  <span> · </span>
  <a href="https://www.facebook.com/CloudOpz-104917804863956">Page</a>
  <span> · </span>
  <a href="https://twitter.com/VuDao81124667">Twitter :stars:</a>
</h3>
