---
title: "Test Post"
date: 2020-05-01T12:31:36+03:00
draft: true
---
 
Подготовил задачу `.gitlab-ci.yml` для тестирования `dotnet core` приложений:

``` yaml
test:
  stage: test
  image: mcr.microsoft.com/dotnet/core/sdk:2.2.301
  before_script:
    - dotnet tool install dotnet-reportgenerator-globaltool --tool-path tools
  script:
    - dotnet test --logger:"junit;LogFilePath=${CI_PROJECT_DIR}/junit/{assembly}-test-result.xml;MethodFormat=Class;FailureBodyFormat=Verbose" --collect:"XPlat Code Coverage"
    - ./tools/reportgenerator "-reports:./**/TestResults/*/coverage.cobertura.xml" "-targetdir:Reports_Coverage" -reportTypes:TextSummary;
    - ./tools/reportgenerator "-reports:./**/TestResults/*/coverage.cobertura.xml" "-targetdir:Reports_Coverage" -reportTypes:Html;
    - cat ./Reports_Coverage/Summary.txt
    #- echo 'End Summary'
  coverage: /Line coverage[\s\S].+%/
  artifacts:
    expire_in: 1 week
    paths:
      - Reports_Coverage/
    reports:
      junit:
        - ${CI_PROJECT_DIR}/junit/*-test-result.xml
```

С шаблоном `junit` столкнулся с проблемой. Если использовать маску `**`, то она не будет сопоставлять пути с прописными символамии (которые были у меня в именах проектов). Поэтому я сделал отдельную директорию для хранения результатов тестов. 

Для включение в `gitlab ci` отображения тестов требуется запустить консоль

`docker-compose exec gitlab gitlab-rails console` и выполнить команду `Feature.enable(:junit_pipeline_view)`
