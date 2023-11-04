# BinSpot
Trading bot for Binance exchange.

Бот BinSpot.
Бот для торговли на Binance spot с использованием стратегии мартингейла (усреднения- увеличения позиции с целью улучшения средней цены входа) и выбором растущей монеты (при работе в LONG, или падающей, при работе в SHORT).

BinSpot выбирает монету, выросшую за выбранный в настройках период времени на указанный процент- сравнивает текущую цену и цену открытия самой дальней выбранной свечи, 
- покупает ее маркет-ордером на указанный объем, 
- выставляет купленные монеты на продажу лимитным sell-ордером по курсу на указанный процент прибыли выше курса покупки,
- выставляет лимитный buy-ордер на покупку этой же монеты по курсу ниже предыдущей покупки на указанный процент (на случай падения курса монеты и уменьшения средней цены входа в сделку).

Затем, в зависимости от того, какой ордер исполнился:
- если исполнился sell-ордер, бот фиксирует прибыль и отменяет buy-ордер для усреднения(если buy-ордер при этом успел исполниться частично или полностью- выставляется sell-ордер), затем снова ищет подходящую пару,
- если исполнился buy-ордер, бот отменяет sell-ордер, выставляет новый sell-ордер уже с новым количеством монет и по новой цене(средняя цена входа + указанный процент прибыли), выставляет новый buy-ордер,
- если buy-ордер исполнился больше, чем наполовину и прошло более 5-ти минут после этого , бот отменяет sell-ордер, и выставляет новый sell-ордер уже с новым количеством монет и по новой цене(средняя цена входа + указанный процент прибыли).

Рекомендуется BinSpot установить на VPS сервер ubuntu 20  или 22, и запускать в SCREEN (чтобы бот не отключался при разрыве SSH-соединения с VPS), настроить telegram-бот и канал, куда будет приходит информация о работе бота. 

- Остановка бота командой:      ctrl+c    (важно!: не останавливайте бот в момент совершения сделок, возможна ошибка записи в базу данных бота).
- В white_list (список пар для работы) можно внести от 1 до нескольких сотен пар, главное, чтобы котируемая валюта была одна: если торгуете к USDT, то пары ETHUSDT, BTCUSDT, XRPUSDT и т.д., если торгуете к BTC, то пары ETHBTC, XRPBTC, DASHBTC и т.д., если к BUSD, то пары с BUSD и т.д.
- При изменении котируемой валюты (с USDT на BTC и т.д.) не забывайте менять и min_order в настройках бота (в USDT min_order должен быть больше 6, а в BTC больше 0.001)
- При работе с парами к USDT активы должны находится на спотовом балансе USDT, если работаете с парами к BTC, активы должны находится на спотовом балансе  BTC.
- Для прокрутки экрана терминала вверх есть команда: ctrl+a, esc и далее стрелка вверх. Для выхода из этого режима: esc, esc.

Для работы BinSpot необходимо использовать BNB для оплаты комиссий биржи (нужно поставить соответствующую галочку в лк binance) и следить за наличием BNB на спот аккаунте.

После закрытия каждой сделки BinSpot:
- отправляет сообщение в telegram-канал, 
- каждую минуту в описание канала отправляет информацию об открытой позиции, 
- в полночь в канал отправляет суточный отчёт о работе. Если не было прибыли за сутки, а также если бот находится в режиме поиска подходящей монеты, то суточный отчёт в telegram не придёт, а в следующий отчёт будет прибавлена предыдущая прибыль за сутки. Точные данные по прибыли наблюдать лучше в лк binance, так как бот показывает приблизительные значения.

Настройки бота:
- fix_perc: процент повышения цены для продажи,
- qty_aver: ввод qty_aver, кратного увеличения объема усредняющего ордера в формате 7 чисел через пробел (например, 1 1.2 1.3 1.4 1.5 1.6 2),
- step_aver: ввод step_aver1, step_aver2, ... step_aver7 шагов изменения цены для выставления усредов,
- limit_aver: разрешенное количество усреднений (от 0 до 30), 
- min_order: минимальный ордер в котируемой валюте (например, в паре ETH/USDT это USDT, ставить не меньше, чем разрешено биржей, например 11),
- min_bal_perc: минимальный процент от депозита, ниже которого BinSpot не будет выставлять усредняющий ордер.
- delta_start: процент изменения цены для входа в сделку, нужно выбрать delta_start1 или delta_start2:  
- delta_start1: для LONG на сколько % должен подняться курс от цены открытия выбранной свечи до текущей для старта (для SHORT пишем со знаком минус, на сколько % должен упасть курс от цены открытия свечи до текущей), 
- delta_start2: для LONG пишем со знаком минус, на сколько % должен упасть курс от цены открытия выбранной свечи до текущей для старта (для SHORT на сколько % должен подняться курс от цены открытия свечи до текущей),
- delta_vol_start- на сколько % должен увеличиться объем торгов последней закрытой свечи по сравнению с объемом выбранной дальней свечи,   
- stop_loss: на сколько % должен упасть курс монеты от средней цены входа для закрытия в минус,
- used_stop_loss: включить использование stop_loss для закрытия в минус (да/нет), если усред-ордер будет частично исполнен, то stop_loss не сработает,  
- pause_after_stop_loss: ставить BinSpot на паузу после срабатывания stop_loss и закрытия позиции по рынку или продолжить работу,
- completed: поставить бота на паузу при закрытии очередной сделки (1-вкл/0-выкл),
- kline_interval: интервал свечей для анализа (1m, 3m, 5m, 15m, 30m, 1h, 2h, 4h, 8h, 12h, 1d, 3d),
- interval_limit: сколько свечей анализируем для выбора цены открытия самой дальней свечи (если следим за изменением цены за последние 3 минуты, то можно выбрать kline_interval 1m и interval_limit 3),  
- super_asset: пара для бесконечной торговли независимо от delta_start, при этом пары из white_list не будут работать (вводится командой -super_asset_add в формате ETHUSDT),
- direction: LONG или SHORT (менять направление работы LONG/SHORT строго рекомендуется в терминале и при отсутствии открытых позиций),
- profit_in: при работе в SHORT можно выбрать частичное накопление прибыли в базовой валюте (по умолчанию только в котируемой), base/quote,
- manual_aver: команда для ручного усреднения по рынку, не дожидаясь цены лимитного усред-ордера, но не сработает, если усред-ордер, выставленный ботом, исполнен частично ('PARTIALLY_FILLED'),
- fix_loss: команда для закрытия открытой позиции по рынку, 
- clear: сброс из базы данных сведений об открытых ботом ордерах,
- clear_profit: сброс из базы данных сведений о прибыли,
- t_sleep: при получении от биржи ошибки о превышении лимита api-запросов, можно выбрать значение паузы в секундах (например: 0.5, 1, 1.5, 3),
- t_sleep_perebor: пауза перед повтором перебора монет для выбора подходящей для старта (по умолчанию 30 сек), 
- white_list: список пар, которые бот будет использовать для анализа и выбора подходящей для открытия сделки (вводится по одной паре командой -w_list_add),
- api_key: открытый api-ключ от биржи с разрешением на спотовую торговлю,
- api_secret: секретный api-ключ от биржи,
- botID: api телеграм бота полученный от @BotFather (пример: 5656544920:AAHrXhjhujhfdf7RPJlheqJXEulBW),
- channelID: ID канала telegram бота для уведомлений, полученное от @userinfobot (пример: -1001656543985),
- tguserid: ID основного user-a телеграм, полученное от @userinfobot (пример: 346549043),  
- licens_key: лицензионный ключ для продления периода работы бота, полученный от разработчика,  
- notif_fix_to_tel: возможность включения функции для отправки сообщений в телеграм-канал о фиксе /on/, или отключения /off/,   
- quoteVolume24hr: минимальный 24-х часовой объем торгов в котируемой валюте (в паре ETH/USDT это USDT, в паре ETH/BTC это BTC), чтобы бот взял пару из white_list в работу.

При выборе направления работы SHORT: 
- активы должны быть в базовой валюте (например: в паре ETH/USDT это ETH),
- min_order: также как и в LONG, указываем в котируемой валюте (в паре ETH/USDT это USDT, ставить больше 11),
- min_bal_perc: указываем ограничивающий процент базовой валюты (ETH),
- delta_start1: указываем с минусом, например -1.7%, 
- delta_start2: указываем плюсовой, например 1.7%, 
- BinBot сначала продает базовую валюту (ETH), затем выставляет FIX ордер на покупку дешевле и выставляет усредняющий SELL ордер на продажу дороже,
- прибыль можно накапливать как в котируемой валюте (USDT), так и частично в базовой (ETH).

Здесь BinSpot представлен для ознакомления и использования в течении пробного периода до 01 ноября 2023 г.
Если Вы хотите увеличить время работы до 1/6/12 месяцев: напишите в телеграм, по данным, указанным при запуске BinSpot.  
При использовании бота на тестовой бирже срок работы не ограничен.

Если хотите испытать BinSpot на спотовой тестовой бирже Binance- 
переходите на:
https://testnet.binance.vision/
оттуда на github, где получите тестовые api-ключи, пропишите их в настройках бота, в used_testnet запишите: да, и экспериментируйте.

BinSpot поставляется по принципу «как есть». Никаких гарантий не прилагается и  не  предусматривается. Вы берете на себя весь риск относительно использования этого бота.
Вы должны понимать, что торговля на криптобиржах сопряжена с повышенным риском, и подходить к управлению рисками со всей ответственностью. 

Пояснения по установке, запуску, настройке бота и телеграм, screen.

Иногда бот может получить от биржи неправильные ответы на api-запросы и выдавать ошибку, поэтому рекомендуется периодически заглядывать в лк binance, и, если бот показывает открытые ордера а в лк binance их нет (или наоборот), нужно использовать команду -clear, чтобы сбросить в боте данные о неактуальных ордерах.
Редко, но бывает, что сервера telegram кратковременно недоступны, и в этот момент сообщение от BinSpot может не доходить в канал бота. 
Для управления ботом на VPS сервере с телефона можно использовать приложение JuiceSSH (или другое для SSH-соединения).

Установка и запуск BinSpot:
- на VPS-сервере ubuntu 20 создайте новую папку, например, BinSpot (   mkdir BinSpot   )
- зайдите в эту папку (   cd BinSpot   )
- перенесите в эту папку файл бота BinSpot-20 (или скачайте с github командой: wget https://github.com/ebot732/BinSpot/releases/download/BinSpot-20/BinSpot-20)
- откройте screen-сессию (например:    screen -S BinSpot   )
- дайте права запуска файлу (команда:    chmod 755 BinSpot-20   )
- запустите BinSpot (команда:    ./BinSpot-20  )
- команда для остановки бота:    ctrl+c
- после запуска бота введите свои параметры: api_key и т.д.
- откорректируйте, при необходимости, настройки
- жмите ENTER и наблюдайте
- для выхода из SCREEN перед закрытием SSH-сессии используйте команду:    ctrl+a, d 
- для входа в screen работающего бота используйте команду:                screen -x BinSpot

Для удобства настройки BinSpot используется телеграм бот, которого нужно сделать админом в телеграм канале. Необходимые данные телеграм бот возьмет из БД BinSpot и будет управляться через чат telegram-Botа. (https://github.com/ebot732/BinSpot/blob/main/README_telegram_uprav_bot.md)

Табличка BinSpot_averaged.xls (https://github.com/ebot732/BinSpot/blob/main/table/BinSpot-18_averaged.xls) показывает приблизительные расчёты усреднений.


            Скриншоты 


![Screenshot](https://github.com/ebot732/BinSpot/blob/main/screenshots/Screenshot_20230513-110159.png)

=======================================================


![Screenshot](https://github.com/ebot732/BinSpot/blob/main/screenshots/Screenshot_20230513-110226.png)

=======================================================


![Screenshot](https://github.com/ebot732/BinSpot/blob/main/screenshots/Screenshot_20230513-110611.png)

=======================================================


![Screenshot](https://github.com/ebot732/BinSpot/blob/main/screenshots/%D0%A1%D0%BD%D0%B8%D0%BC%D0%BE%D0%BA%20%D1%8D%D0%BA%D1%80%D0%B0%D0%BD%D0%B0%20%D0%BE%D1%82%202023-05-13%2010-52-59.png)

=======================================================


![](./screenshots/Spot_work1.png)    

==========================   

время |BUY/SELL| статус |количество| пара | сумма | цена | id ордера на бирже    
![](./screenshots/Spot_work1_start_order.png)   
Стартовый и усредняющие ордера в консоли выделены зеленым цветом.

![](./screenshots/Spot_work1_FIX_order.png)   
Ордер (FIX) для закрытия сделки по стратегии выделен фиолетовым цветом.

![](./screenshots/Spot_work1_string_status_bot.png)   
Строка состояния бота выделена желтым цветом.





