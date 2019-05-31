# Installing Laravel Valet on macOS

[installing laravel valet on macOS](https://aregsar.com/blog/2019/installing-laravel-valet-on-macOS)

## Introduction

In this post I will detail the installation and running of Laravel Valet on macOS using the PHP Composer package manager.

Note: Laravel Valet has been ported to Ubuntu Linux. I will provide links for installing Valet on Linux or Windows Subsystem for Linux on Windows 10.

Laravel Valet is a package that consists of a daemon that runs in the background and serves up Laravel sites through an Nginx proxy server. Valet automatically installs Nginx, if it is not already installed, via the Homebrew package manager for macOS.

Valet also installs Dnsmasq via Homebrew. Dnsmasq acts a local DNS server intercepting and routing URIs with the `.test` domain to our local Nginx server.

The URI's that Valet serves will be of the form `<project-name>.test` where the project name in the URL specifies which Laravel project will be served.

> Note: we can configure a different domain then `.test`. You can check out the documentation for Laravel Valet for further info. `https://laravel.com/docs/5.8/valet`.

https://laravel.com/docs/5.8/valet#installation

## Installing Valet via Composer

To install Valet we can run:

`composer global require laravel/valet`

the global `composer.json` file is located at:

`~/.composer/vendor/bin/composer.json`

So requiring valet globally installs valet at:

`~/.composer/vendor/bin/vendor/laravel/valet/valet`

and adds the symlinks:

`/usr/local/bin/valet` -> `~/.composer/vendor/bin/vendor/laravel/valet/valet`

and

`~/.composer/vendor/bin/vendor/bin/valet` -> `~/.composer/vendor/bin/vendor/laravel/valet/valet`

So we have to make sure either of these symlinks is added to the path env variable.

Composer also creates the configuration directory:

`~/.config/valet/`

We can check the version:

`valet --version`

## Configure Valet with NGinx ad Dnsmaq

`valet install`

On macOS uses Homebrew to install nginx and dnsmasq

#setup websites

Go to any directory where our laravel project directories reside and run:

`valet park`

This will serve any project using the URI `<projectname>.test` in the web browser












