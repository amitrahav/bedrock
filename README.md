# [Bedrock](https://roots.io/bedrock/) on aws ElastickBeanstalk multi containers Docker
[![Packagist](https://img.shields.io/packagist/v/roots/bedrock.svg?style=flat-square)](https://packagist.org/packages/roots/bedrock)
[![Build Status](https://img.shields.io/travis/roots/bedrock.svg?style=flat-square)](https://travis-ci.org/roots/bedrock)

Bedrock is a modern WordPress stack that helps you get started with the best development tools and project structure.

Much of the philosophy behind Bedrock is inspired by the [Twelve-Factor App](http://12factor.net/) methodology including the [WordPress specific version](https://roots.io/twelve-factor-wordpress/).

## Features

* Better folder structure (adding site/ folder to separate php application from deployment configurations)
* Dependency management with [Composer](http://getcomposer.org)
* Easy WordPress configuration with environment specific files
* Environment variables with [Dotenv](https://github.com/vlucas/phpdotenv)
* Autoloader for mu-plugins (use regular plugins as mu-plugins)
* Enhanced security (separated web root and secure passwords with [wp-password-bcrypt](https://github.com/roots/wp-password-bcrypt))


See a complete working example in the [roots-example-project.com repo](https://github.com/roots/roots-example-project.com).

## Software Requirements

* PHP >= 5.6
* Composer - [Install](https://getcomposer.org/doc/00-intro.md#installation-linux-unix-osx)

## Deployment Requirements

* Aws account
* Configured pipeline with:
  * CodeCommit.
  * CodeBuild on a docker environment can run composer.
  * Elastic Container Service with good php-fpm image - to run composer and php-fpm on.
  * Elastic Beanstalk on a multi docker environment.

## Installation

1. Create a new project in a new folder for your project:

  `composer create-project roots/bedrock your-project-folder-name`

2. Update environment variables in `.env`  file - mostly for local development:

  * `RDS_DB_NAME` - Database name
  * `RDS_USERNAME` - Database user
  * `RDS_PASSWORD` - Database password
  * `RDS_HOSTNAME` - Database host
  * `WP_ENV` - Set to environment (`development`, `staging`, `production`)
  * `WP_HOME` - Full URL to WordPress home (http://example.com)
  * `WP_SITEURL` - Full URL to WordPress including subdirectory (http://example.com/wp)
  * `AUTH_KEY`, `SECURE_AUTH_KEY`, `LOGGED_IN_KEY`, `NONCE_KEY`, `AUTH_SALT`, `SECURE_AUTH_SALT`, `LOGGED_IN_SALT`, `NONCE_SALT`

  If you want to automatically generate the security keys (assuming you have wp-cli installed locally) you can use the very handy [wp-cli-dotenv-command][wp-cli-dotenv]:

      wp package install aaemnnosttv/wp-cli-dotenv-command

      wp dotenv salts regenerate

  Or, you can cut and paste from the [Roots WordPress Salt Generator][roots-wp-salt].

  On ElasticBeanstalk multi containers environment, env vars are transferd throgh RDS data tier and custome env vars you can insert on ebstalk->environment->config->softwer configurations.
  so on this version of bedrock - the env vars are ElasticBeanstalk multi containers Ready

3. Add theme(s) in `web/app/themes` as you would for a normal WordPress site.

4. Set your site vhost document root to `/path/to/site/web/` (`/path/to/site/current/web/` if using deploys)

5. Access WP admin at `http://example.com/wp/wp-admin`

## Deploys

### defining CodePipeline:

* selecting codeCommit as a source (or any other source supported and available).
* running codeBuild with buildspec.yml - IMPORTANT! configure buildspec to run on a php-fpm image with composer install on it.
* configure ElasticBeanstalk to use multi containers docker environment.
* push code to codeCommit to start deployment process.

### How does it works?

1. codeBuild install all composer dependencies and create artifact that include all files necessary to run the software.
2. ElasticBeanstalk deploys all those files into a host.
3. php-fpm docker copy all site files and run php-fpm.
4. nginx docker copy all site and config files and run nginx so it can serve all existing files.

### Static files
[amazon web services wp plugin](https://wordpress.org/plugins/amazon-web-services/) is required by default by composer.json, it's critical to configure it.
it's recommended to serve files with aws cloudFront, but also possible with simple aws S3 with hosting permission.

## Documentation

Bedrock documentation is available at [https://roots.io/bedrock/docs/](https://roots.io/bedrock/docs/).

BeanStalk multi containers docker documentation is available at [http://docs.aws.amazon.com/elasticbeanstalk/latest/dg/create_deploy_docker_ecs.html](http://docs.aws.amazon.com/elasticbeanstalk/latest/dg/create_deploy_docker_ecs.html).

## Contributing

Contributions are welcome from everyone. We have [contributing guidelines](https://github.com/roots/guidelines/blob/master/CONTRIBUTING.md) to help you get started.

## Community

Keep track of development and community news.

* Participate on the [Roots Discourse](https://discourse.roots.io/)
* Follow [@rootswp on Twitter](https://twitter.com/rootswp)
* Read and subscribe to the [Roots Blog](https://roots.io/blog/)
* Subscribe to the [Roots Newsletter](https://roots.io/subscribe/)
* Listen to the [Roots Radio podcast](https://roots.io/podcast/)

[roots-wp-salt]:https://roots.io/salts.html
[wp-cli-dotenv]:https://github.com/aaemnnosttv/wp-cli-dotenv-command
