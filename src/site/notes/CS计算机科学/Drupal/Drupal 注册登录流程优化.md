---
{"dg-publish":true,"permalink":"/CS计算机科学/Drupal/Drupal 注册登录流程优化/","noteIcon":"","created":"2024-06-22T22:30:28.476+08:00","updated":"2024-05-06T11:39:54.000+08:00"}
---


作者: 游鱼思

---

有时候真的很想吐槽，许多开源软件的用户体验太差了，例如Drupal的登录注册流程就很反直觉、反人性。

---

## Simple Password Reset

## Email Registration

子模块：Email Registration Username

[Email Registration | Drupal.org](https://www.drupal.org/project/email_registration)

## Mail Login

[Mail Login | Drupal.org](https://www.drupal.org/project/mail_login)

- **Enable login by email address:** This option enables login by email address.
- **Override login form:** This option allows you to override the login form username title/description.
- **Login by email address only:** This option disables login by username and forces login by email address only.
- **Override login form:** This option allows you to override the login form username title/description.
- **Login form username/email address label:** Override the username field title.
- **Login form username/email address description:** Override the username field description.
- **Login form email address only label:** Override the username field title.
- **Login form email address only description:** Override the username field description.

## Password Reset Landing Page (PRLP)

User Registration Password

## 通过接口注册登录

请查看 [Drupal 对外提供API](数据交互/Drupal%20对外提供API.md)
