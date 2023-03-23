# ML System Design Doc - Остап Бендер v0.1 [RU]
## Дизайн ML системы - Отелит Подбор Арендатора №1

*Шаблон ML System Design Doc от телеграм-канала [Reliable ML](https://t.me/reliable_ml)*   

- Рекомендации по процессу заполнения документа (workflow) - [здесь](https://github.com/IrinaGoloshchapova/ml_system_design_doc_ru/blob/main/ML_System_Design_Doc_Workflow.md). 
- [Шаблон дизайн-дока](https://github.com/IrinaGoloshchapova/ml_system_design_doc_ru).
    
> ## Термины и пояснения
> - Товар - помещение, которое сдаётся в аренду
> - Клиент - компания или предприниматель, который ищет или искал помещение в аренду и уже обращался в компанию в его поисках, но не обязательно заключил сделку
> - Сделка - подписанный и оплаченный договор аренды
> - Лид - Клиент, который еще не был проверен Менеджером и не получил первичную подборку коммерческих предложений
> - ИММО - Итерация по моделированию машинного обучения (ИММО). Состоит из шагов поиска данных, их обработки, подготовки к машинному обучению, само машинное обучение и анализ полученных результатов

### 1. Цели и предпосылки 
#### 1.1. Зачем идем в разработку продукта?  

- Бизнес-цель: увеличить среднюю конверсию из лида в успешную сделку для арендаторов до 3-4% с текущих 1-2%. Промежуточные цели: поднять % перехода из лида в сделку. Тут сложно определить целевую величину, так как сейчас сделки создаются не предсказуемо без правил. В разработке стоит задача сделать автоматизацию создания сделок каждый раз, когда клиенту отправляется КП. С её внедрением показатель автоматически вырастет. На стадии деплоя этой разработки придется наблюдать за этой метрикой снова и отталкиваться уже от неё. Сейчас в сделку переходят 15% лидов.
- Почему станет лучше, чем сейчас, от использования:  система сможет прогнозировать сделку лучше, чем это делает человек. Клиенту делается предложение 2-4 помещений из 100+. В этом случае человек точно упускает варианты и даже не предлагает их. Машина способна поднять это качество и помогать менеджеру делать подборку ничего не пропуская и с большей вероятностью в успехе.

#### 1.2. Бизнес-требования и ограничения  

- Логика работы ML:  Машинное обучение работает по принципу запрос-ответ. Из CRM системы поступает запрос с данными о Клиенте и Менеджере (признаками для предикта). В ответ система выдаёт ответ с номерами айди товаров которые на её взгляд приведут к сделке у данного менеджера с данным клиентом.
- Бизнес-требования: Запрос-ответ должен происходить не более 10 секунд. В идеале 2-3.
- Успешная модель: успешной будет модель, которая предлагает на уровне опытного менеджера такие товары, которые будут выгодны менеджеру и интересны клиенту для заключения Сделки. 
- Возможные пути развития проекта: Шаг 1 (Остап v1.0). Убрать всё не подходящее. Модель классификации товаров. Товары классифицируются на подходящий и неподходящий. Версии модели отличаются уровнем качества. Подверсии удобством и устойчивостью.  Шаг 2 (Остап v2.0). Ранжировать подходящее. Рекомендательная система. Товары расставляются в порядке вероятности заключения сделки. Версии модели отличаются по уровню качества. Подверсии - интерпретируемостью модели, устойчивостью. Шаг 3(Остап v3.0). Рекомендательная система подбора арендатора. Товарам рекомендуются клиенты. Модель оценивает сумму и вероятность сделки. Версии отличаются точностью. Подверсии - удобством и устойчивостью.
- Среда: разработка на мощностях компании в её удалённом сервере.
- Стэк-технологии: Разработку проводить на открытом ПО и коде.
- Данные: Используется база данных CRM компании. В данных для обучения может быть недостаточное количество успешных сделок или собраны не все необходимые признаки. Также отсутствует информация о фактических предложениях и отказе в них, которые не были записаны в соварнную сделку. Данные собраны за 8 лет работы и старые сделки могут путать систему. Полнота и достоверность данных лучше с 2020 года и хуже до этого периода. Актуализировать данные можно, но это займёт время.
- Исполнители: Проект выполняется мной, поэтому могут быть ограничения связанные с нехваткой опыта/знаний

#### 1.3. Что входит в скоуп проекта/итерации, что не входит   

- Что мы ожидаем от конкретной итерации: проверка достаточности данных, построение модели на локальной машине на имеющихся данных. Уточнение признаков, метрик и их целевых значение. 
- На закрытие каких БТ подписываемся в данной итерации: проверка работоспособности модели на имеющихся данных в юпитере на локальной машине без выкатывания в прод или развертывания бэка на сервере. Собираем данные, обрабатываем, обучаем модели, смотрим результат.
- Что не будет закрыто: в этой итерации не будет закрыто требование успеха проекта - повышение конверсии, поскольку мы это пока не измеряем. Пока не выкатываем в рабочий кластер. Не определяем нужный стэк и ресурсы. Не занимаемся настройкой фронта на строне б24.
- Результат с точки зрения качества кода и воспроизводимости решения: в данный момент рабочая тетрадь юпитера, в котором подняты первичные данные и сделана МЛ, способная делать предикты для новых клиентов. Внутри комментарии касаемо сделанных шагов, как нужно обработать данные в базе и того, что нужно сделать для безотказной работы.
- Описание планируемого технического долга (что оставляем для дальнейшей продуктивизации): настройка пайплайна, версирование моделей и данных. Описать ограничения и требования. Определить стэк.

#### 1.4. Предпосылки решения  

- Описание всех общих предпосылок решения, используемых в системе: Используем данные о сделках с гранулярностью: клиент-товар-статус сделки. Признаками выступают заполненные в сделках данные. Горизонтом прогноза выступает сделка. 

### 2. Методология    

#### 2.1. Постановка задачи  

- Что делаем с технической точки зрения: в V1.0 MVP - это задача классификации. В V2.0 и V3.0 - рекомендательные системы.

#### 2.2. Блок-схема решения  

- [Блок-схема](https://github.com/nikita4099/otelit/blob/8110929d721e79ae79a238e29ae8034a794282bf/%D0%91%D0%BB%D0%BE%D0%BA%20%D1%81%D1%85%D0%B5%D0%BC%D0%B0%20%D0%9E%D1%81%D1%82%D0%B0%D0%BF.PNG) для MVP и пилота с ключевыми этапами решения задачи.

#### 2.3. Этапы решения задачи.  

- Этап 1 - подготовка бейзлайна. 

Шаг 1. Сбор данных, отбор признаков, анализ. 
Скачиваем данные о сделках из CRM системы. Загружаем их в тетрадь юпитера. При отборе проверяем заполненность данных. По итогу количественные признаки должны быть количественными, качественные качественными. 

Целевая переменная
| Название данных  | Есть ли данные в компании (если да, название источника/витрин) | Требуемый ресурс для получения данных (какие роли нужны) | Проверено ли качество данных (да, нет) |
| ------------- | ------------- | ------------- | ------------- |
| Статус сделки (успешная или проваленная) | CRM.Сделки  | DE | + |

Признаки: 
| Название данных  | Есть ли данные в компании (если да, название источника/витрин) | Требуемый ресурс для получения данных (какие роли нужны) | Проверено ли качество данных (да, нет) |
| ------------- | ------------- | ------------- | ------------- |
| Менеджер | CRM. Айди менеджера  | DE | + | - айди пользователя используем не как число, а как класс. Модел должна учитывать, что у разных менеджеров разное представление подходит/не подходит.
| Размах компании со слов клиента или по мнению менеджера | Лид.Размах компании | DE | - |
| Чем занимается арендатор | Компания/лид.Чем занимается | DE | - |
| Минимальная площадь аренды со слов клиента | Компания/лид.Площади от | DE | - |
| Максимальная площадь аренды со слов клиента | Компания/лид.Площади до | DE | - |
| Площадь вакантного помещения | Товар свободный.Площадь | DE | + |
| Прайс цена вакантного помещения за аренду в месяц | Товар свободный.Цена за всё помещение | DE | + |
| Сколько клиент в принципе может платить аренды | Лид.Цена за всю площадь-максимум, руб. | DE | - |
| Приоритетный берег для аренды со слов клиента | Лид. Какой берег интересует? | DE | - |
| Какой район интересует интересует клиента с его слов | Лид. Какой район интересует? | DE | - |

MVP: тетрать юпитера с загруженными признаками.

Шаг 2. Обработка данных, подготовка к обучению.

- Количественные признаки: Проверяем количественные данные на выбросы, убираем строки, в которых данные похожи на ошибку или могут мешать системе учиться на бОльшей части данных. Также удаляем строки, где количественные признаки не заполнены. 
- Качественные признаки: заполняем пропуски как неизвестное значение. Приводим в вид OHE. Учитываем то, что клиенты занимаются несколькими бизнесами, ищут помещения в нескольких районах и нескольких берегах.
- Группируем уволенных менеджеров как один менеджер
- Описание формирования выборки для обучения, тестирования и валидации: за основу берутся сделки, с подкреплённым товаром и заполненными признаками. Так как данные накопленные за разный промежуток времени, часть сделок уже не актуальна, то для пилота данные перемешаем и запустим трейн на 75% с кросс-валидацией на 5 батчах. Оставшиеся 25% будем использовать для тестирования. Идея заключается в том, что решение модели будет использоваться не для прогнозирования сделок, а для прогнозирования провалов. В этом ключе проверку модели в бою можно будет провести довольно быстро. Использовать лиды сегодняшнего дня и экспертно проверить, насколько хорошо или прекрасно модель определила злокачественные варианты. Если качество будет слабым, то проверяем данные уже тщательно и повторяем процедуру.

MVP: тетрадь упитера с утвержденным перечнем признаков и подготовленный к обучению моделей.

Шаг 3. Поиск и обучение модели с необходимым для бейзлайна уровнем метрик. 

- определяем метрики качества модели: например, высокий recall, потому что нам важно не потерять качественные варианты. 
- проверяем несколько моделей. Желательно штук 5-7.
- находим через гиперпараметры лучшую модель. 

В случае, если в результате обучения качество модели не удаётся поднять до уровня бейзлайна необходимо провести дополнительное изучение базы данных на предмет потенциальных признаков, играющих важную роль в подборе.

MVP: тетрадь юпитера с определённой лучшей моделью и её параметрами, отвечающая заявленными критериям качества

Шаг 4. Анализ результатов. Проверка значимости признаков.
- Интерпретации результата: Проверка на адекватность. Если клиент снял помещение 40 м2, хотя искал 100-200 - это должно игнорироваться моделью. Если большинство пекарен снимают помещение с ценой до 40000 рублей, то помещение с ценой 80000 будет помечаться скорей отрицательно. 

MVP: обученная модель соблюдает бизнес-логику

Шаг 5. - Развертывание ИММО согласно бизнес требованиям заказчика. Готовим стэк согласно требованиям заказчика, переносим туда наш бейзлайн и добиваемся того, чтобы модель могла исполняться согласно заявленной логике. Настраиваем интеграцию с тестовой базой, разворачиваем интерфейс, настраиваем пайплайны, редактируем базу данных. 

- Описание техники достижения поставленных бизнес-требований: Для улучшения скорости в систему МЛ периодически (надо решить как часто) фоново попадают обновления базы данных Товаров и Менеджеров и все остальные необходимые данные. Таким образом пакет данных из боевой базы уменьшается кратно и отправляются только данные о Клиенте и ID менеджера. Пересчёт эмбедингов делать чаще раза в месяц не эффективно, так как бизнес-цикл происходит на этом промежутке. В ответ МЛ должна давать батчи с парой Товар-Класс, который на стороне боевой базы используется как фильтр предложений. 

Проверяем выбранные прокси-метрики:
1. % созданных сделок, которые наша модель разметила как негативные. Бейзлайн = 50%.
2. % успешных сделок, которые наша модель разметила как негативные. Бейзлайн = 50%.
3. % неверно убранных предложений на 10 рандомных лидах в неделю проверенные экспертом. Бейзлайн = 50%. Если подходит 15 из 100, то модель должна ошибочно негативно разметить не больше 8 из них. 8/15 = 53%.

MVP: модель работает с тестовой средой с заявленным качеством согласно бизнес-требованиям.

Шаг 6. - Считаем системные требования к модели. Проводим расчёт вычислительной мощности необходимой для работы системы. Заполняем 4-й раздел дизайн-дока.

MVP: составлены системные требования для модели

Этап 2. Создание пилота.

Шаг 7. - Утверждение метрик пилота. Определяем финальные метрики, прокси-метрики и технические метрики, при достижении которых модель будет запущена в деплой. Цели подтверждает заказчик, дата-саентист и дата-инженер.

MVP: определены целевые метрики на основании имеющихся данных и согласованные с бизнесом

Шаг 8. - Подготовка модели к деплою. Проводим обогощение модели данными, улучшаем предобработку и подбор параметров модели до тех пор, пока не удастся достичь заявленные цели.

MVP: модель, удовлетворяющая заявленным метрикам

Шаг 9. - Деплой. Выкатываем модель в рабочую среду и проводим тестирование на достижение метрик и отказоустойчивости согласно требованиям.


#### 2.4 Какие могут быть риски и что планируем с этим делать. 

Неверная интерпретируемость данных: проверка данных
Низкое качество модели: добавление признаков
Недостающие ресурсы: купим

#### 3.1. Способ оценки пилота  
  
- Краткое описание предполагаемого дизайна и способа оценки пилота: проверяем модель на 10 клиентах. Проводим оценку % потерянных предложений. Считаем какое количество предложений на мнение эксперта подходили клиенту и смотрим, какое количество из них система отметила как неподходящие.

#### 3.2. Что считаем успешным пилотом
  
Формализованные в пилоте метрики оценки успешности: При потере не более 3 из 15 (FN = 20%) вариантов модель считаем успешной.

### 4. Внедрение 

#### 4.1. Архитектура решения   
  
- Блок схема и пояснения: сервисы, назначения, методы API `Data Scientist` : попробуем на Docker сделать. ML как сервис. Через API запросы идут. Но на данном этапе делаем тетрадку юпитера с импортом данных из csv. 
  
#### 4.2. Описание инфраструктуры и масштабируемости 
  
- Какая инфраструктура выбрана и почему `Data Scientist`: надо изучить варианты. Пока никакая
- Плюсы и минусы выбора `Data Scientist` : - 
- Почему финальный выбор лучше других альтернатив `Data Scientist` : - 
  
#### 4.3. Требования к работе системы  
  
- SLA, пропускная способность и задержка `Data Scientist`  : -
  
#### 4.4. Безопасность системы  
  
- Потенциальная уязвимость системы `Data Scientist`  : тут могу сказать, что всё может не заработать из-за того, что данные очень растянуты по времени и в идеале обучение делать не на старых данных, а на срезе данных. Какие были помещения в момент заключения сделки кроме сданного помещения. С точки зрения используемого программного обеспечения или устарения данных, то риски пока оценить сложно, да и возможно не нужно.
  
#### 4.5. Безопасность данных   
  
- Нет ли нарушений GDPR и других законов `Data Scientist`  : нет
  
#### 4.6. Издержки  
  
- Расчетные издержки на работу системы в месяц `Data Scientist` : минимальные. Если используем только открытый код, то кластер у нас есть, его ресурсов должно хватать для пилота
  
#### 4.5. Integration points  
  
- Описание взаимодействия между сервисами (методы API и др.) `Data Scientist` : пока не известно
  
#### 4.6. Риски  
  
- Описание рисков и неопределенностей, которые стоит предусмотреть `Data Scientist` : пока не известно

> ### Материалы для дополнительного погружения в тему  
> - [Шаблон ML System Design Doc [EN] от AWS](https://github.com/eugeneyan/ml-design-docs) и [статья](https://eugeneyan.com/writing/ml-design-docs/) с объяснением каждого раздела  
> - [Верхнеуровневый шаблон ML System Design Doc от Google](https://towardsdatascience.com/the-undeniable-importance-of-design-docs-to-data-scientists-421132561f3c) и [описание общих принципов его заполнения](https://towardsdatascience.com/understanding-design-docs-principles-for-achieving-data-scientists-53e6d5ad6f7e).
> - [ML Design Template](https://www.mle-interviews.com/ml-design-template) от ML Engineering Interviews  
> - Статья [Design Documents for ML Models](https://medium.com/people-ai-engineering/design-documents-for-ml-models-bbcd30402ff7) на Medium. Верхнеуровневые рекомендации по содержанию дизайн-документа и объяснение, зачем он вообще нужен  
> - [Краткий Canvas для ML-проекта от Made with ML](https://madewithml.com/courses/mlops/design/#timeline). Подходит для верхнеуровневого описания идеи, чтобы понять, имеет ли смысл идти дальше.  
