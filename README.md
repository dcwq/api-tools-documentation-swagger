# Swagger Documentation Provider for Laminas API Tools

[![Build Status](https://travis-ci.com/laminas-api-tools/api-tools-documentation-swagger.svg?branch=master)](https://travis-ci.com/laminas-api-tools/api-tools-documentation-swagger)
[![Coverage Status](https://coveralls.io/repos/github/laminas-api-tools/api-tools-documentation-swagger/badge.svg?branch=master)](https://coveralls.io/github/laminas-api-tools/api-tools-documentation-swagger?branch=master)

## Introduction

This module provides Laminas API Tools the ability to show API documentation through a
[Swagger UI](http://swagger.io/).

The Swagger UI is immediately accessible after enabling this module at the URI path `/api-tools/swagger`.

In addition to providing the HTML UI, this module also plugs into the main Laminas API Tools documentation
resource (at the path `/api-tools/documentation`) in order to allow returning a documentation
payload in the `application/vnd.swagger+json` media type; this resource is what feeds the Swagger
UI. You can access this representation by passing the media type `application/vnd.swagger+json` for
the `Accept` header via the path `/api-tools/documentation/:module/:service`.

## Requirements
  
Please see the [composer.json](composer.json) file.

## Installation

Run the following `composer` command:

```console
$ composer require laminas-api-tools/api-tools-documentation-swagger
```

Alternately, manually add the following to your `composer.json`, in the `require` section:

```javascript
"require": {
    "laminas-api-tools/api-tools-documentation-swagger": "^1.2"
}
```

And then run `composer update` to ensure the module is installed.

Finally, add the module name to your project's `config/application.config.php` under the `modules`
key:

```php
return [
    /* ... */
    'modules' => [
        /* ... */
        'Laminas\ApiTools\Documentation\Swagger',
    ],
    /* ... */
];
```

> ### laminas-component-installer
>
> If you use [laminas-component-installer](https://github.com/laminas/laminas-component-installer),
> that plugin will install api-tools-documentation-swagger as a module for you.

## Routes

### /api-tools/swagger

Shows the Swagger UI JavaScript application.

### Assets: `/api-tools-documentation-swagger/`

Various CSS, images, and JavaScript libraries required to deliver the Swagger UI client
application.

## Configuration

### System Configuration

The following is required to ensure the module works within a Laminas and/or Laminas API Tools-enabled
application:

```php
namespace Laminas\ApiTools\Documentation\Swagger;

return [
    'router' => [
        'routes' => [
            'api-tools' => [
                'child_routes' => [
                    'swagger' => [
                        'type' => 'segment',
                        'options' => [
                            'route'    => '/swagger',
                            'defaults' => [
                                'controller' => SwaggerUi::class,
                                'action'     => 'list',
                            ],
                        ],
                        'may_terminate' => true,
                        'child_routes' => [
                            'api' => [
                                'type' => 'segment',
                                'options' => [
                                    'route' => '/:api',
                                    'defaults' => [
                                        'action' => 'show',
                                    ],
                                ],
                                'may_terminate' => true,
                            ],
                        ],
                    ],
                ],
            ],
        ],
    ],

    'service_manager' => [
        'factories' => [
            SwaggerViewStrategy::class => SwaggerViewStrategyFactory::class,
        ],
    ],

    'controllers' => [
        'factories' => [
            SwaggerUi::class => SwaggerUiControllerFactory::class,
        ],
    ],

    'view_manager' => [
        'template_path_stack' => [
            'api-tools-documentation-swagger' => __DIR__ . '/../view',
        ],
    ],

    'asset_manager' => [
        'resolver_configs' => [
            'paths' => [
                __DIR__ . '/../asset',
            ],
        ],
    ],

    'api-tools-content-negotiation' => [
        'accept_whitelist' => [
            'Laminas\ApiTools\Documentation\Controller' => [
                0 => 'application/vnd.swagger+json',
            ],
        ],
        'selectors' => [
            'Documentation' => [
                ViewModel::class => [
                    'application/vnd.swagger+json',
                ],
            ],
        ],
    ],
];
```

### Module Documentation

Some information needed in the Swagger documentation cannot be retreieved from the standard API documentation module
but is taken from the module's `documentation.config.php` file instead. Extend these files with the following keys
to complete the Swagger JSON output:

```php
<?php
return [
    // these fields are directly merged into the Swagger JSON output and can provide/override
    // any property that is supported by the Swagger 2.0 Specification. See https://swagger.io/docs/specification/2-0/basic-structure/
    'Laminas\\ApiTools\\Documentation\\Swagger\\Api' => [
        'info' => [
            'title' => 'My API',
            'description' => '',
        ],
        'securityDefinitions' => [
            'basic-auth' => [
                'type' => 'basic',
            ],
            'application-http' => [
                'type' => 'apiKey',
                'in' => 'header',
                'name' => 'Authorization',
            ],
        ],
    ],
    // Swagger properties merged into each Service definition
    'Api\\V1\\Rest\\Some\\Controller' => [
        // reference to a security definition and specify the requird scope (for oauth2)
        'security' => 'basic-auth',
        'scope' => [],
    ],
    'Api\\V1\\Rest\\Other\\Controller' => [
        'security' => 'application-http',
        'scope' => [],
        'collection' => [
            // describe supported query parmeters extracted from the 'collection_query_whitelist' config
            'query' => [
                'q' => [
                    'type' => 'string',
                    'description' => 'Search term for filtering',
                ],
            ],
        ],
    ],
];
```

## Laminas Events

### Listeners

#### Laminas\ApiTools\Documentation\Swagger\Module

This listener is attached to the `MvcEvent::EVENT_RENDER` event at priority `100`.  Its purpose is
to conditionally attach a view strategy to the view system in cases where the controller response is
a `Laminas\ApiTools\Documentation\Swagger\ViewModel` view model (likely selected as the
content-negotiated view model based off of `Accept` media types).

## Laminas Services

### View Models

#### Laminas\ApiTools\Documentation\Swagger\ViewModel

This view model is responsible for translating the available `Laminas\ApiTools\Documentation` models
into Swagger-specific models, and further casting them to arrays for later rendering as JSON.
