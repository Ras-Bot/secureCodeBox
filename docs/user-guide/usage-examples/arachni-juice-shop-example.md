# Scanning modern Single Page Application like OWASP JuiceShop using arachni

Scanning for Vulnerabilities in OWASP JuiceShop using arachni and the secureCodeBox API.

## Introduction

JuiceShop poses some problems for many scanners, as it is written as a single page application. This means that the scanner needs to be able to execute JavaScript to spider the application.

Luckily arachni is intended for exactly this case!

## Testing the setup

To start of we should test weather the secureCodeBox and JuiceShop is properly set up. To do this we can start a minimal scan which will finish relatively quickly. This is done by setting the Page Limit to `1` so that arachni will only test the first page of the application.

`PUT http://localhost:8080/box/processes/arachni_webapplicationscan`

```json
[
    {
        "name": "Arachni Quick Test Scan",
        "location": "http://juice-shop:3000/",
        "attributes": {
            "ARACHNI_DOM_DEPTH_LIMIT": 5,
            "ARACHNI_DIR_DEPTH_LIMIT": 5,
            "ARACHNI_PAGE_LIMIT": 1,
            "ARACHNI_EXCLUDE_PATTERNS": [],
            "ARACHNI_SCAN_METHODS": "*"
        }
    }
]
```

This scan should finish in about a minute and should return a couple of findings.

## Full JuiceShop Arachni Config

A full arachni scan config for JuiceShop includes optimisations to maximise the amount of vulnerabilities discovered and optimise the scan time.

This is done by configuring the following parameters:

1.  Increase Scan Depth (DOM_DEPTH, DIR_DEPTH and PAGE_LIMIT) to more sensible values for the application.
2.  Exclude non relevant endpoints. This greatly decreases the scan time as arachni doesn't have to waste time scanning non relevant resources like static css/js files or socket.io endpoints.
3.  Extending the spider by routes we know about but arachni wont be able to find via its spider because the are not linked anywhere in the application.
4.  Provide arachni with the ability to log into the application. This is often the hardest part of the scan config. Without it the scanner will not be able access restricted pages, so that the scanner will only scan a subset of the application. In this case the login is performed using the `login-script` plugin of arachni which lets us script how to to the login. This script is included in the default `docker-compose` setup of the secureCodeBox. This script is located in `plugins/arachni-login-scripts/login_juice_shop.rb`. You can add custom login scripts for your application into the same directory, the will get mounted into the arachni container.

`PUT http://localhost:8080/box/processes/arachni_webapplicationscan`

```json
[
    {
        "name": "In Depth Arachni Scan",
        "location": "http://juice-shop:3000/",
        "attributes": {
            "ARACHNI_DOM_DEPTH_LIMIT": 50,
            "ARACHNI_DIR_DEPTH_LIMIT": 10,
            "ARACHNI_PAGE_LIMIT": 100,
            "ARACHNI_EXCLUDE_PATTERNS": [
                ".*socket\\.io.*",
                ".*node_modules.*",
                ".*public/images.*",
                ".*/css.*"
            ],
            "ARACHNI_SCAN_METHODS": "*",
            "ARACHNI_EXTEND_PATH": ["/#/administration"],
            "ARACHNI_LOGIN_SCRIPT_FILENAME": "login_juice_shop.rb"
        }
    }
]
```