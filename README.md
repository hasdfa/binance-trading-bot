# Binance Trading Bot

[![GitHub version](https://img.shields.io/github/package-json/v/chrisleekr/binance-trading-bot)](https://github.com/chrisleekr/binance-trading-bot/releases)
[![Build](https://github.com/chrisleekr/binance-trading-bot/workflows/Push/badge.svg)](https://github.com/chrisleekr/binance-trading-bot/actions?query=workflow%3APush)
[![CodeCov](https://codecov.io/gh/chrisleekr/binance-trading-bot/branch/master/graph/badge.svg)](https://codecov.io/gh/chrisleekr/binance-trading-bot)
[![Docker pull](https://img.shields.io/docker/pulls/chrisleekr/binance-trading-bot)](https://hub.docker.com/r/chrisleekr/binance-trading-bot)
[![GitHub contributors](https://img.shields.io/github/contributors/chrisleekr/binance-trading-bot)](https://github.com/chrisleekr/binance-trading-bot/graphs/contributors)
[![MIT License](https://img.shields.io/github/license/chrisleekr/binance-trading-bot)](https://github.com/chrisleekr/binance-trading-bot/blob/master/LICENSE)

> Автоматизированный торговый бот Binance со скользящей стратегией покупки/продажи

---

[![ko](https://img.shields.io/badge/lang-한국어-brightgreen.svg)](https://github.com/chrisleekr/binance-trading-bot/blob/master/README.ko.md)
[![中文](https://img.shields.io/badge/lang-中文-blue.svg)](https://github.com/chrisleekr/binance-trading-bot/blob/master/README.zh-cn.md)

Это тестовый проект. Я просто тестирую свой код.

## Предупреждения

** Я не могу гарантировать, сможете ли вы зарабатывать деньги или нет.**

** Так что используйте его на свой страх и риск! Я не несу ответственности за любые потери или трудности
понесенные прямо или косвенно при использовании этого кода. Читать
[отказ от ответственности](#disclaimer) перед использованием этого кода.**

** Перед обновлением бота обязательно запишите в заметку последнюю цену покупки. Он может потерять конфигурацию или записи последней цены покупки.**

## Как это работает

### Trailing Grid Trade Buy/Sell Bot

Этот бот использует концепцию скользящего ордера на покупку/продажу, который позволяет следить за падением/ростом цены.

> Скользящие стоп-приказы
> С концепцией скользящих стоп-ордеров вы можете ознакомиться в [Официальном документе Binance] (https://www.binance.com/en/support/faq/360042299292)
>
> TL;DR
> Размещайте ордера на фиксированную стоимость или процент при изменении цены. Используя эту функцию, вы можете покупать по минимально возможной цене при покупке вниз и продавать по максимально возможной цене при продаже вверх.

- Бот поддерживает несколько ордеров на покупку/продажу в зависимости от конфигурации.
- Бот может отслеживать несколько символов. Все символы будут контролироваться в секунду.
- Бот использует MongoDB для предоставления постоянной базы данных. Однако он не использует последнюю версию MongoDB для поддержки 32-битной версии Raspberry Pi. Используемая версия MongoDB
   3.2.20, предоставленный [apcheamitru](https://hub.docker.com/r/apcheamitru/arm32v7-mongo).
- Бот протестирован/работает с Linux и Raspberry Pi 4 32bit. Другие платформы не тестировались.

#### Купить Сигнал

Бот будет постоянно отслеживать монету на основе конфигурации торговли сетки.

Для сделки по сетке № 1 бот размещает ордер STOP-LOSS-LIMIT на покупку, когда текущая цена достигает минимальной цены. Если текущая цена постоянно падает, то бот отменит предыдущий ордер и заменит новый ордер STOP-LOSS-LIMIT с новой ценой.

После сделки по сетке № 1 бот будет отслеживать МОНЕТУ на основе последней цены покупки.

- Бот не будет размещать ордер на покупку сетки #1, если у него достаточно монет (обычно более 10 долларов США) для продажи, когда будет достигнута триггерная цена для продажи.
- Бот удалит последнюю цену покупки, если оценочное значение меньше порога удаления последней цены покупки.

##### Купить Сценарий

Скажем, если конфигурации торговли сетки покупки установлены, как показано ниже:

- Количество сеток: 2
- Сетки
  | No# | Процент срабатывания | Процент стоп-цены     | Процент предельной цены | USDT |
  | --- | -------------------  | --------------------- | ----------------------  | ---- |
  | 1   | 1                    | 1.05                  | 1.051                   | 50   |
  | 2   | 0.8                  | 1.03                  | 1.031                   | 100  |

Чтобы упростить понимание, я буду использовать «$» в качестве символа USDT. Для простого расчета я не беру в расчет комиссию. В реальной торговле количество может быть другим.

Ваша первая торговля по сетке на покупку настроена следующим образом:

- Сетка №#: 1
- Процент срабатывания: 1
- Процент остановки: 1,05 (5,00%)
- Предельный процент: 1,051 (5,10%)
- Максимальная сумма покупки: $50

И рынок, как показано ниже:

- Текущая цена: $105
- Самая низкая цена: $100
- Начальная цена: $100

Когда текущая цена падает до самой низкой цены (100 долларов США) и ниже, чем ограниченная цена ATH (All-Time High), если она включена, бот размещает новый ордер STOP-LOSS-LIMIT на покупку.

- Стоп-цена: 100 долларов * 1,05 = 105 долларов.
- Лимитная цена: 100$ * 1,051 = 105,1$
- Количество: 0,47573

Предположим, что рынок меняется следующим образом:

- Текущая цена: $95

Затем бот будет следить за падением цены и размещать новый ордер STOP-LOSS-LIMIT, как показано ниже:

- Стоп-цена: 95$ * 1,05 = 99,75$
- Лимитная цена: $95 * 1,051 = $99,845
- Количество: 0,5

Предположим, что рынок меняется следующим образом:

- Текущая цена: $100

Затем бот выполнит 1-ю покупку за монету. Последняя цена покупки будет записана как «$99,845». Приобретенное количество будет «0,5».

Как только монета будет куплена, бот начнет отслеживать сигнал на продажу и в то же время следить за следующей сеткой, торгующей на покупку.

Ваша 2-я сетка, торгующая на покупку, настроена следующим образом:

- Сетка №: 2
- Текущая цена последней покупки: $99,845
- Процент срабатывания: 0,8 (20%)
- Процент остановки: 1,03 (3,00%)
- Предельный процент: 1,031 (3,10%)
- Максимальная сумма покупки: $100

И если текущая цена постоянно падает до $79.876 (на 20% ниже), то бот выставит новый ордер STOP-LOSS-LIMIT для торговли монетой во 2-й сетке.

Предположим, что рынок меняется следующим образом:

- Текущая цена: $75

Затем бот будет следить за падением цены и размещать новый ордер STOP-LOSS-LIMT, как показано ниже:

- Стоп-цена: 75$ * 1,03 = 77,25$
- Лимитная цена: 75$ * 1,031 = 77,325$
- Количество: 1,29

Предположим, что рынок меняется следующим образом:

- Текущая цена: $78

Затем бот выполнит вторую покупку за монету. Последняя цена покупки будет автоматически пересчитана, как показано ниже:

- Окончательная цена последней покупки: (50 $ + 100 $)/(0,5 МОНЕТЫ + 1,29 МОНЕТЫ) = 83,80 $

##### Подробная информация о настройке покупки

Подробный документ по конфигурации покупки доступен здесь.

[https://github.com/chrisleekr/binance-trading-bot/wiki/Buy-Scenario](https://github.com/chrisleekr/binance-trading-bot/wiki/Buy-Scenario)

### Сигнал на продажу

Если баланса достаточно для продажи и в боте зафиксирована последняя цена покупки, то бот начнет отслеживать сигнал продажи сетки №1. Как только текущая цена достигнет цены срабатывания сетки № 1, бот разместит ордер STOP-LOSS-LIMIT на продажу. Если текущая цена постоянно растет, то бот отменит предыдущий ордер и заменит новый ордер STOP-LOSS-LIMIT с новой ценой.

- Если у бота нет записи о последней цене покупки, бот не продаст монету.
- Если монета стоит меньше порога удаления последней цены покупки, то бот удалит последнюю цену покупки.
- Если монета не стоит минимальной номинальной стоимости, то бот не выставит ордер.

#### Сценарий продажи

Скажем, если конфигурации торговли сетки продажи установлены, как показано ниже:

- Количество сеток: 2
- Сетки
  | No# | Процент срабатывания | Процент стоп-цены     | Процент предельной цены | USDT |
  | --- | -------------------  | --------------------- | ----------------------  |------------------------- |
  | 1st | 1.05                 | 0.97                  | 0.969                   | 0.5                      |
  | 2nd | 1.08                 | 0.95                  | 0.949                   | 1                        |

В отличие от покупки, конфигурация продажи будет использовать процент от количества. Если вы хотите продать все свое количество монет, просто настройте его как «1» (100%).

Из последних действий по покупке у вас теперь есть следующие балансы:

- Текущее количество: 1,79
- Текущая цена последней покупки: $83,80.

Ваша первая торговля по сетке для продажи настроена следующим образом:

- Сетка №1
- Процент срабатывания: 1,05
- Процент стоп-цены: 0,97
- Процент предельной цены: 0,969
- Процент суммы продажи: 0,5

Предположим, что рынок меняется следующим образом:

- Текущая цена: $88

Поскольку текущая цена выше цены срабатывания продажи (87,99 долл. США), бот разместит новый ордер STOP-LOSS-LIMIT на продажу.

- Стоп-цена: $88 * 0,97 = $85,36.
- Лимитная цена: $88 * 0,969 = $85,272
- Количество: 0,895

Предположим, что рынок меняется следующим образом:

- Текущая цена: $90

Затем бот будет следить за ростом цены и размещать новый ордер STOP-LOSS-LIMIT, как показано ниже:

- Цена стопа: 90$ * 0,97 = 87,30$
- Лимитная цена: 90$ * 0,969 = 87,21$
- Количество: 0,895

Предположим, что рынок меняется следующим образом:

- Текущая цена: $87

Затем бот выполнит первую продажу монеты. Затем бот теперь будет ждать 2-й триггерной цены продажи (83,80 долл. США * 1,08 = 90,504 долл. США).

- Текущее количество: 0,895
- Текущая цена последней покупки: $83,80.

Предположим, что рынок меняется следующим образом:

- Текущая цена: $91

Тогда текущая цена (91 доллар США) выше, чем 2-я триггерная цена продажи (90,504 доллара США), бот разместит новый ордер STOP-LOSS-LIMIT, как показано ниже:

- Стоп-цена: $91 * 0,95 = $86,45.
- Лимитная цена: 91$ * 0,949 = 86,359$
- Количество: 0,895

Предположим, что рынок меняется следующим образом:

- Текущая цена: $100

Затем бот будет следить за ростом цены и размещать новый ордер STOP-LOSS-LIMT, как показано ниже:

- Стоп-цена: 100 долларов * 0,95 = 95 долларов.
- Лимитная цена: 100 $ * 0,949 = 94,9 $.
- Количество: 0,895

Предположим, что рынок меняется следующим образом:

- Текущая цена: $94

Затем бот выполнит вторую продажу монеты.

Конечная прибыль будет

- 1-я продажа: $94,9 * 0,895 = $84,9355
- 2-я продажа: 87,21 доллара * 0,895 = 78,05295 доллара.
- Итоговая прибыль: $162 (прибыль 8%).

##### Углубленная настройка продажи

Подробный документ по конфигурации покупки доступен здесь.

[https://github.com/chrisleekr/binance-trading-bot/wiki/Sell-Scenario](https://github.com/chrisleekr/binance-trading-bot/wiki/Sell-Scenario)

### [Features](https://github.com/chrisleekr/binance-trading-bot/wiki/Features)

- Ручная торговля
- Конвертировать небольшие остатки в BNB
- Торговать всеми символами
- Мониторинг нескольких монет одновременно
- Остановить потери
- Ограничить покупку по цене ATH
- Сеточная торговля на покупку/продажу
- Интегрирован с техническим анализом TradingView

### Frontend + WebSocket

Интерфейс на основе React.js, взаимодействующий через веб-сокет:

- Мониторинг списка монет с сигналами покупки/продажи/открытыми ордерами
- Просмотр остатков на счетах
- Просмотр открытых/закрытых сделок
- Управление глобальными / символьными настройками
- Удалить кеши, которые не отслеживаются
- Ссылка на общедоступный URL
- Поддержка добавления на главный экран
- Безопасный интерфейс

## Environment Parameters

Use environment parameters to adjust parameters. Check `/config/custom-environment-variables.json` to see list of available environment parameters.

Or use the frontend to adjust configurations after launching the application.

## How to use

1. Create `.env` file based on `.env.dist`.

   | Environment Key                | Description                                                               | Sample Value                                                                                        |
   | ------------------------------ | ------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------- |
   | BINANCE_LIVE_API_KEY           | Binance API key for live                                                  | (from [Binance](https://binance.zendesk.com/hc/en-us/articles/360002502072-How-to-create-API))      |
   | BINANCE_LIVE_SECRET_KEY        | Binance API secret for live                                               | (from [Binance](https://binance.zendesk.com/hc/en-us/articles/360002502072-How-to-create-API))      |
   | BINANCE_TEST_API_KEY           | Binance API key for test                                                  | (from [Binance Spot Test Network](https://testnet.binance.vision/))                                 |
   | BINANCE_TEST_SECRET_KEY        | Binance API secret for test                                               | (from [Binance Spot Test Network](https://testnet.binance.vision/))                                 |
   | BINANCE_SLACK_ENABLED          | Slack enable/disable                                                      | true                                                                                                |
   | BINANCE_SLACK_WEBHOOK_URL      | Slack webhook URL                                                         | (from [Slack](https://slack.com/intl/en-au/help/articles/115005265063-Incoming-webhooks-for-Slack)) |
   | BINANCE_SLACK_CHANNEL          | Slack channel                                                             | "#binance"                                                                                          |
   | BINANCE_SLACK_USERNAME         | Slack username                                                            | Chris                                                                                               |
   | BINANCE_LOCAL_TUNNEL_ENABLED   | Enable/Disable [local tunnel](https://github.com/localtunnel/localtunnel) | true                                                                                                |
   | BINANCE_LOCAL_TUNNEL_SUBDOMAIN | Local tunnel public URL subdomain                                         | binance                                                                                             |
   | BINANCE_AUTHENTICATION_ENABLED | Enable/Disable frontend authentication                                    | true  |
   | BINANCE_AUTHENTICATION_PASSWORD | Frontend password                                                        | 123456 |
   | BINANCE_LOG_LEVEL               | Logging level. [Possible values described on `bunyan` docs.](https://www.npmjs.com/package/bunyan#levels) | ERROR |

   *A local tunnel makes the bot accessible from the outside. Please set the subdomain of the local tunnel as a subdomain that only you can remember.*
   *You must change the authentication password; otherwise, it will be configured as the default password.*

2. Launch/Update the bot with docker-compose

   Pull latest code first:

   ```bash
   git pull
   ```

   If want production/live mode, then use the latest build image from DockerHub:

   ```bash
   docker-compose -f docker-compose.server.yml pull
   docker-compose -f docker-compose.server.yml up -d
   ```

   Or if want development/test mode, then run below commands:

   ```bash
   docker-compose up -d --build
   ```

3. Open browser `http://0.0.0.0:8080` to see the frontend

   - When launching the application, it will notify public URL to the Slack.
   - If you have any issue with the bot, you can check the log to find out what happened with the bot. Please take a look [Troubleshooting](https://github.com/chrisleekr/binance-trading-bot/wiki/Troubleshooting)

### Install via Stackfile

1. In [Portainer](https://www.portainer.io/) create new Stack

2. Copy content of `docker-stack.yml` or upload the file

3. Set environment keys for `binance-bot` in the `docker-stack.yml`

4. Launch and open browser `http://0.0.0.0:8080` to see the frontend

## Screenshots

| Password Protected | Frontend Mobile |
| ------------------ | --------------- |
| ![Password Protected](https://user-images.githubusercontent.com/5715919/133920104-49d1b590-c2ba-46d7-a294-eb6b24b459f5.png) | ![Frontend Mobile](https://user-images.githubusercontent.com/5715919/137472107-4059fcdf-5174-4282-81af-80cea5b269a0.png) |

| Setting | Manual Trade |
| ------- | ------------ |
| ![Setting](https://user-images.githubusercontent.com/5715919/127318581-4e422ac9-b145-4e83-a90d-5c05c61d6e2f.png) | ![Manual Trade](https://user-images.githubusercontent.com/5715919/127318630-f2180e1b-3feb-48fa-a083-4cb7f90f743f.png) |

| Frontend Desktop  | Closed Trades |
| ----------------- | ------------- |
| ![Frontend Desktop](https://user-images.githubusercontent.com/5715919/137472148-7be1e19b-3ce5-4d5a-aa28-18c55b3b48aa.png) | ![Closed Trades](https://user-images.githubusercontent.com/5715919/137472190-a4c6ef0f-3399-44bb-852f-eedb7c67d629.png) |

### Sample Trade

| Chart                                                                                                          | Buy Orders                                                                                                          | Sell Orders                                                                                                          |
| -------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------- |
| ![Chart](https://user-images.githubusercontent.com/5715919/111027391-192db300-8444-11eb-8df4-91c98d0c835b.png) | ![Buy Orders](https://user-images.githubusercontent.com/5715919/111027403-36628180-8444-11eb-91dc-f3cdabc5a79e.png) | ![Sell Orders](https://user-images.githubusercontent.com/5715919/111027411-4b3f1500-8444-11eb-8525-37f02a63de25.png) |

## Changes & Todo

Please refer
[CHANGELOG.md](https://github.com/chrisleekr/binance-trading-bot/blob/master/CHANGELOG.md)
to view the past changes.

- [ ] Develop simple setup screen for secrets
- [ ] Allow to execute stop-loss before buy action - [#299](https://github.com/chrisleekr/binance-trading-bot/issues/299)
- [ ] Improve sell strategy with conditional stop price percentage based on the profit percentage - [#94](https://github.com/chrisleekr/binance-trading-bot/issues/94)
- [ ] Add sudden drop buy strategy - [#67](https://github.com/chrisleekr/binance-trading-bot/issues/67)
- [ ] Manage setting profiles (save/change/load?/export?) - [#151](https://github.com/chrisleekr/binance-trading-bot/issues/151)
- [ ] Improve notifications by supporting Apprise - [#106](https://github.com/chrisleekr/binance-trading-bot/issues/106)
- [ ] Support cool time after hitting the lowest price before buy - [#105](https://github.com/chrisleekr/binance-trading-bot/issues/105)
- [ ] Reset global configuration to initial configuration - [#97](https://github.com/chrisleekr/binance-trading-bot/issues/97)
- [ ] Support multilingual frontend - [#56](https://github.com/chrisleekr/binance-trading-bot/issues/56)
- [ ] Non linear stop price and chase function - [#246](https://github.com/chrisleekr/binance-trading-bot/issues/246)
- [ ] Support STOP-LOSS configuration per grid trade for selling - [#261](https://github.com/chrisleekr/binance-trading-bot/issues/261)

## Donations

If you find this project helpful, feel free to make a small
[donation](https://github.com/chrisleekr/binance-trading-bot/blob/master/DONATIONS.md)
to the developer.

## Acknowledgments

- [@d0x2f](https://github.com/d0x2f)
- And many others! Thanks guys!

## Contributors

Thanks to all contributors :heart: [Click to see our heroes](https://github.com/chrisleekr/binance-trading-bot/graphs/contributors)

## Disclaimer

I give no warranty and accepts no responsibility or liability for the accuracy or the completeness of the information and materials contained in this project. Under no circumstances will I be held responsible or liable in any way for any claims, damages, losses, expenses, costs or liabilities whatsoever (including, without limitation, any direct or indirect damages for loss of profits, business interruption or loss of information) resulting from or arising directly or indirectly from your use of or inability to use this code or any code linked to it, or from your reliance on the information and material on this code, even if I have been advised of the possibility of such damages in advance.

**So use it at your own risk!**
