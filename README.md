# Opensource Bitrix Order

Более подробную информацию по функционалу, а так же просто рассуждения на тему читайте по ссылке:
https://verstaem.com/bitrix/opensource-order/

Здесь, в README.md краткая выжимка из статьи касательно функционала, но без объяснения причин.

## Как установить?

Установка доступна тремя разными способами:

1\. Установка через композер

Чтобы пакет скачался в нужную папку в файле composer.json укажите путь до папки bitrix. Обратите внимание на блок extra:
```json
{
    "name": "your/project",
    "authors": [
        {
            "name": "Alexander Shubin",
            "email": "alorian@yandex.ru"
        }
    ],
    "require": {},
    "extra": {
        "bitrix-dir": "./"
    }
}
```
Путь до папки bitrix нужно прописывать относительно файла composer.json. Например если файл composer.json лежит в /local/libs,
то нужно прописать "bitrix-dir": "../../bitrix". По дефолту установщик считает, что файл composer.json лежит в document_root.
Если не указать корректный bitrix-dir, то будет создана папка bitrix/modules/opensource.order/ рядом с composer.json.

После того как прописали правильный bitrix-dir выполните: 
```bash
$ composer require alorian/bxorder
```

После выполнения команды откройте список модулей маркетплейс в админке /bitrix/admin/partner_modules.php?lang=ru, если
bitrix-dir был указан корректно, то вы увидите строку с модулем opensource.order. Нажмите "Установить" в выпадающем меню.

2\. Установка из маркетплейс

Перейдите по ссылке https://marketplace.1c-bitrix.ru/solutions/opensource.order/ и установите решение как обычно.
Если страница не открывается, то возможно решение еще на модерации.

3\. Ручная установка

Скачайте архив https://github.com/alorian/bxorder/archive/master.zip и самостоятельно распакуйте его содержимое
в папку модулей битрикса -- /bitrix/modules, либо /local/modules.

В папке модулей у вас должна быть папка opensource.order, а не bxorder-master, папку bxorder-master которая лежит
в архиве необходимо переименовать. Таким образом полный путь до файла include.php у вас должен 
быть /bitrix/modules/opensource.order/include.php, либо /local/modules/opensource.order/include.php

После распаковки архива откройте список модулей маркетплейс в админке /bitrix/admin/partner_modules.php?lang=ru, 
найдите строку с модулем opensource.order и нажмите "Установить" в выпадающем меню

---

После установки любым из указанных способов разместите компонент opensource:order на нужной странице.

## Как использовать?

Что вам нужно сделать как программисту для интеграции верстки? В самом простом случае вам всего лишь нужно сформировать 
форму (html тэг form) в шаблоне компонента, которая при отправке передаст на сервер пять переменных:

1\. person_type_id. Переменная которая содержит тип плательщика.

2\. properties[]. Массив переменных со свойствами заказа. Например, если у свойства символьный код — FIO, то атрибут 
name у инпута ставьте properties[FIO]. Если переменная множественная то ставьте name=properties[FIO][]

3\. delivery_id. В самом простом случае это просто input типа radio, у которого атрибут name=delivery_id

4\. pay_system_id. Так же как и с доставкой, просто radio инпут, только атрибут name=pay_system_id

5\. save. Если переменная save=y, то компонент сохранит заказ. Во всех остальных случаях компонент просто обновит 
данные в объекте заказа и отдаст шаблон.

Да, всё настолько просто. Формируете форму с этими пятью переменными и вы великолепны. Это далеко не всё что можно 
сделать с помощью опенсорсного компонента. Но даже в самых сложных шаблонах суть останется прежней. 
Оформление заказа это не магия, просто обычная форма в браузере, просто чуть больше полей чем в обратной связи.

## Что передается из компонента в шаблон?

Компонент формирует объект заказа и объект с коллекцией ошибок. Массив $arResult не используется.

Чтобы получить доступ к объекту заказа и коллекции ошибок в файле result_modifier.php шаблона вставьте в начало следующий 
код:
```php
<?php
/**
 * @var OpenSourceOrderComponent $component
 */
$component = &$this->__component;
$order = $component->order;
$errorCollection = $component->errorCollection;
```

В шаблоне компонента желательно использовать предварительно сформированный в result_modifier массив $arResult (см. демо шаблоны). 
Но если вы  по каким либо причинам хотите получить прямой доступ к объекту заказа, то можете сделать это так:
```php
<?php
if (!defined('B_PROLOG_INCLUDED') || B_PROLOG_INCLUDED !== true) {
    die();
}

/** @var OpenSourceOrderComponent $component */
echo $component->order->getPrice();
var_dump($component->errorCollection);
```

## Куда отправлять аякс запросы?

В компоненте есть три готовых аякс метода, все они лежат в файле [ajax.php](https://github.com/alorian/bxorder/blob/master/install/components/order/ajax.php).
Все они работают с помощью ["нового механизма аякс"](https://verstaem.com/ajax/new-bitrix-ajax/)

1. searchLocation. Возвращает найденные местоположения по любой строке вида "москва", "санкт" и т.д.
2. calculateDeliveries. Возвращает массив со стоимостью доставок
3. saveOrder. Сохранение массива аяксом. Для работы нужно сериализовать все поля формы и отправить. 

## Как дорабатывать компонент под проект?

Во первых, вы всегда можете отправить pull request. Если там будет код, который пригодится многим, то pull request будет
влит в основную ветку. Таким образом вы будете пользоваться оригинальным компонентом, без каких либо доработок.

Во вторых, вы можете использовать систему событий битрикс. Наиболее полезным я думаю будет событие [OnSaleOrderBeforeSaved](https://dev.1c-bitrix.ru/api_d7/bitrix/sale/events/order_saved.php).

В третьих, вы можете использовать [возможность наследования компонентов](https://dev.1c-bitrix.ru/learning/course/index.php?COURSE_ID=43&LESSON_ID=2028).
В наследованном компоненте пишете любой нужный под проект функционал, и при этом не теряете совместимость с оригинальным компонентом.
Шаблон компонента у вас в любом случае будет свой.

## Куда отправлять ошибки и предложения?

Компонент более менее протестирован. Но возможно вы найдете какие-либо баги, в этом случае создайте [issue](https://github.com/alorian/bxorder/issues) в этом репозитории. 
Предложения по доработке тоже можно писать туда.