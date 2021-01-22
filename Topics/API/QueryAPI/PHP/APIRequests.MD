# Glimesh API Requests With PHP

This is a basic tutorial for communicating with the Glimesh API using PHP. This guide assumes you know basic PHP and have an access token from a user. You can use a client ID if you do not yet have an access token. You will also need an Apache or NGINX server with PHP running. Having basic HTML, CSS, and GraphQL knowledge would also be helpful.
> Need a refresh? This tutorial explains the basics of GraphQL. https://graphql.org/learn/schema/


## Creating the Project

We need to create a basic project. I will be using an Apache server in this tutorial.

> Here's a LAMP stack tutorial by Digital Ocean for Ubuntu 18.04 [External Link](https://www.digitalocean.com/community/tutorials/how-to-install-linux-apache-mysql-php-lamp-stack-ubuntu-18-04)

With the Apache server with PHP running, we will now start creating a basic project.


## Starting Up

In your create or add the following directories and files to get the project started. If you already have a project started, you may skip to another part of this tutorial for what you're looking for.

> assets/css/bootstrap.min.css [Download File]()
> assets/css/custom.css
> assets/inc/glimesh.php
> index.php


## glimesh.php

Now open the "glimesh.php" file and paste this at the top of your file.

```php
<?php
session_start();
define('CLIENT_ID', 'qwerty12345');
define('CLIENT_SECRET', 'qwerty12345');
$OAUTH_URL = 'https://glimesh.tv/oauth/authorize';
$TOKEN_URL = 'https://glimesh.tv/api/oauth/token';
$API_URL = 'https://glimesh.tv/api';
```
> Glimesh application client id and secret goes here, this information is in your application and is meant to be kept secret! 


Next, we will now make an action to "login" to our application with Glimesh API.

```php
if(get('action') == 'login') {
  $params = array(
    'client_id' => CLIENT_ID,
    'redirect_uri' => 'http://localhost/',
    'response_type' => 'code',
    'scope' => 'public chat'
  );
  header('Location: ' . $OAUTH_URL . '?' . http_build_query($params));
  die();
}
if(get('code')) {
  $token = api_request($TOKEN_URL, array(
    'grant_type' => 'authorization_code',
    'client_id' => CLIENT_ID,
    'client_secret' => CLIENT_SECRET,
    'redirect_uri' => 'http://localhost/',
    'code' => get('code')
  ));
  $_SESSION['access_token'] = $token->access_token;
  header('Location: ' . $_SERVER['PHP_SELF']);
}
```
> When our action to login with glimesh is called it will send a request for an access token when the "code" parameter has a value.


We need a few functions to complete the "glimesh.php" file, here's what is left to add:

```php
function api_request($url, $post = FALSE, $headers = array()) {
  $ch = curl_init($url);
  curl_setopt($ch, CURLOPT_IPRESOLVE, CURL_IPRESOLVE_V4);
  curl_setopt($ch, CURLOPT_RETURNTRANSFER, TRUE);
  $response = curl_exec($ch);
  if($post) curl_setopt($ch, CURLOPT_POSTFIELDS, http_build_query($post));
  $headers[] = 'Accept: application/json';
  if(session('access_token')) $headers[] = 'Authorization: Bearer ' . session('access_token');
  curl_setopt($ch, CURLOPT_HTTPHEADER, $headers);
  $response = curl_exec($ch);
  return json_decode($response);
}
function get($key, $default = NULL) {
  return array_key_exists($key, $_GET) ? $_GET[$key] : $default;
}
function session($key, $default = NULL) {
  return array_key_exists($key, $_SESSION) ? $_SESSION[$key] : $default;
}
function graphql_query($endpoint, $query, $token = null) : array {
  $headers = ['Content-Type: application/json', 'User-Agent: minimal GraphQL client'];
  if(null !== $token) {
    $headers[] = "Authorization: Bearer " . $token;
  }
  if(false === $data = @file_get_contents($endpoint, false, stream_context_create([
    'http' => [
      'method' => 'POST',
      'header' => $headers,
      'content' => json_encode(['query' => $query]),
    ]
  ]))) {
    $error = error_get_last();
    throw new \ErrorException($error['message'], $error['type']);
  }
  return json_decode($data, true);
}
?>
```
> These functions will send our requests and queries to the Glimesh API. Credits to dunglas on Github for the "graphql_query" function, I slightly modified.

The full file can be found here:

```php
<?php
session_start();
define('CLIENT_ID', 'qwerty12345');
define('CLIENT_SECRET', 'qwerty12345');
$OAUTH_URL = 'https://glimesh.tv/oauth/authorize';
$TOKEN_URL = 'https://glimesh.tv/api/oauth/token';
$API_URL = 'https://glimesh.tv/api';
if(get('action') == 'login') {
  $params = array(
    'client_id' => CLIENT_ID,
    'redirect_uri' => 'http://localhost/',
    'response_type' => 'code',
    'scope' => 'public chat'
  );
  header('Location: ' . $OAUTH_URL . '?' . http_build_query($params));
  die();
}
if(get('code')) {
  $token = api_request($TOKEN_URL, array(
    'grant_type' => 'authorization_code',
    'client_id' => CLIENT_ID,
    'client_secret' => CLIENT_SECRET,
    'redirect_uri' => 'http://localhost/',
    'code' => get('code')
  ));
  $_SESSION['access_token'] = $token->access_token;
  header('Location: ' . $_SERVER['PHP_SELF']);
}
if(get('action') == 'logout') {
  session_destroy();
  unset($_SESSION['access_token']);
  header('Location: http://localhost/');
  die();
}
function api_request($url, $post = FALSE, $headers = array()) {
  $ch = curl_init($url);
  curl_setopt($ch, CURLOPT_IPRESOLVE, CURL_IPRESOLVE_V4);
  curl_setopt($ch, CURLOPT_RETURNTRANSFER, TRUE);
  $response = curl_exec($ch);
  if($post) curl_setopt($ch, CURLOPT_POSTFIELDS, http_build_query($post));
  $headers[] = 'Accept: application/json';
  if(session('access_token')) $headers[] = 'Authorization: Bearer ' . session('access_token');
  curl_setopt($ch, CURLOPT_HTTPHEADER, $headers);
  $response = curl_exec($ch);
  return json_decode($response);
}
function get($key, $default = NULL) {
  return array_key_exists($key, $_GET) ? $_GET[$key] : $default;
}
function session($key, $default = NULL) {
  return array_key_exists($key, $_SESSION) ? $_SESSION[$key] : $default;
}
function graphql_query($endpoint, $query, $token = null) : array {
  $headers = ['Content-Type: application/json', 'User-Agent: minimal GraphQL client'];
  if (null !== $token) {
    $headers[] = "Authorization: Bearer " . $token;
  }
  if (false === $data = @file_get_contents($endpoint, false, stream_context_create([
    'http' => [
      'method' => 'POST',
      'header' => $headers,
      'content' => json_encode(['query' => $query]),
    ]
  ]))) {
    $error = error_get_last();
    throw new \ErrorException($error['message'], $error['type']);
  }
  return json_decode($data, true);
}
?>
```


## index.php

```php
<?php include 'assets/inc/glimesh.php'; ?>
<!DOCTYPE HTML>
<html>
  <head>
    <title>Glimesh API | PHP Example</title>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1, user-scalable=no" />
    <link rel="stylesheet" href="assets/css/bootstrap.min.css" />
    <link rel="stylesheet" href="assets/css/custom.css" />
  </head>
  <body style="background-color: #060818;color: #FFFFFF;">
    <div class="center-screen panel">
      
    </div>
  </body>
</html>
```

