# Аутентификация с помощью G-Suite

С помощью [федерации удостоверений](../../concepts/users/identity-federations.md) вы можете использовать [G-Suite](https://gsuite.google.com/) от Google для аутентификации в облаке.

Чтобы настроить аутентификацию:

1. [Начните создавать SAML-приложение](#configure-sso-gsuite-start).
1. [Создайте федерацию в облаке](#create-federation).
1. [Добавьте сертификаты в федерацию](#add-certificate).
1. [Получите ссылку для входа в консоль](#get-link).
1. [Завершите создание SAML-приложения](#configure-sso-gsuite-finish).
1. [Добавьте пользователей в облако](#add-users).
1. [Протестируйте аутентификацию](#test-auth).

## Перед началом {#before-you-begin}

Для того, чтобы воспользоваться инструкциями в этом разделе, вам понадобятся:
1. Роль [`admin`](../../concepts/access-control/roles.md#admin) или [`resource-manager.clouds.owner`](../../concepts/access-control/roles.md#owner) в облаке.
1. Активированный домен, для которого вы будете настраивать SAML-приложение в G-Suite.

## Начните создавать SAML-приложение {#configure-sso-gsuite-start}

Прежде чем вы создадите федерацию в облаке, вам необходимо получить сведения о поставщике удостоверений (IdP), которым является SAML-приложение в G-Suite:

1. Зайдите в [консоль администратора в G-Suite](https://admin.google.com/).
1. Нажмите на иконку **Приложения**.
1. Нажмите на карточку **SAML-приложения**.
1. Нажмите на кнопку добавления приложения (значок **+** в правом нижнем углу страницы).
1. Внизу открывшегося окна нажмите **Добавить свое приложение**.
1. На странице **Сведения о поставщике услуг идентификации Google** написаны данные сервера IdP. Не закрывайте это окно, эти данные необходимо будет ввести при [создании федерации](#create-federation) и [добавлении сертификата](#add-certificate).

## Создайте федерацию в облаке {#create-federation}

{% list tabs %}

- Консоль управления

    1. Откройте страницу каталога в [консоли управления]({{ link-console-main }}).
    1. В меню слева выберите вкладку **Федерации**.
    1. Нажмите **Создать федерацию**.
    1. Задайте имя федерации. Имя должно быть уникальным в каталоге.
    1. При необходимости добавьте описание.
    1. В поле **Время жизни cookie** укажите время, в течении которого браузер не будет требовать у пользователя повторной аутентификации.
    1. В поле **IdP Issuer** скопируйте ссылку, указанную в поле **Идентификатор объекта** на странице **Сведения о поставщике услуг идентификации Google** в G-Suite. Это ссылка в формате:
        ```
        https://accounts.google.com/o/saml2?idpid=<ID SAML-приложения>
        ```
    1. В поле **Ссылка на страницу для входа в IdP** скопируйте ссылку, указанную в поле **URL Системы единого входа** на странице **Сведения о поставщике услуг идентификации Google** в G-Suite. Это ссылка в формате:
        ```
        https://accounts.google.com/o/saml2/idp?idpid=<ID SAML-приложения>
        ```
    1. Включите опцию **Автоматически создавать пользователей**, чтобы новый пользователь автоматически добавлялся в облако после успешной аутентификации. Эта опция упрощает процесс заведения пользователей, но у пользователя будет только роль `resource-manager.clouds.member` и он не сможет выполнять никаких операций с ресурсами в этом облаке. Исключение — те ресурсы, на которые назначены роли системной группе `allUsers` или `allAuthenticatedUsers`.

        Если эту опцию не включать, то пользователь, которого не добавили в облако, не сможет войти, даже если пройдет аутентификацию на вашем сервере. Таким образом можно создать <q>белый список</q> пользователей, которым разрешено пользоваться Яндекс.Облаком.

- CLI

    {% include [cli-install](../../../_includes/cli-install.md) %}

    {% include [default-catalogue](../../../_includes/default-catalogue.md) %}

    1. Посмотрите описание команды создания федерации:

        ```
        $ yc iam federation create --help
        ```

    1. Создайте федерацию:

        ```
        $ yc iam federation create --name my-federation \
            --auto-create-account-on-login \
            --cookie-max-age 12h \
            --issuer "https://accounts.google.com/o/saml2?idpid=C03xolm0y" \
            --sso-binding POST \
            --sso-url "https://accounts.google.com/o/saml2/idp?idpid=C03xolm0y"
        ```

        Где:

        * `name` — имя федерации. Имя должно быть уникальным в каталоге.
        * `auto-create-account-on-login` — флаг, активирующий автоматическое создание новых пользователей в облаке после аутентификации на IdP-сервере. Эта опция упрощает процесс заведения пользователей, но созданному таким образом пользователю по умолчанию назначается только роль `resource-manager.clouds.member`: он не сможет выполнять никаких операций с ресурсами в этом облаке. Исключение — те ресурсы, на которые назначены роли системной группе `allUsers` или `allAuthenticatedUsers`.

            Если эту опцию не включать, то пользователь, которого не добавили в облако, не сможет войти в консоль управления, даже если пройдет аутентификацию на вашем сервере. В этом случае вы можете управлять белым списком пользователей, которым разрешено пользоваться Яндекс.Облаком.
        * `cookie-max-age` — время, в течении которого браузер не должен требовать у пользователя повторной аутентификации.
        * `issuer` — идентификатор IdP-сервера, на котором должна происходить аутентификация.

            Скопируйте сюда ссылку, указанную в поле **Идентификатор объекта** на странице **Сведения о поставщике услуг идентификации Google** в G-Suite. Это ссылка в формате:

            ```
            https://accounts.google.com/o/saml2?idpid=<ID SAML-приложения>
            ```
        * `sso-url` — URL-адрес страницы, на которую браузер должен перенаправить пользователя для аутентификации.

            Скопируйте сюда ссылку, указанную в поле **URL Системы единого входа** на странице **Сведения о поставщике услуг идентификации Google** в G-Suite. Это ссылка в формате:

            ```
            https://accounts.google.com/o/saml2/idp?idpid=<ID SAML-приложения>
            ```
        * `sso-binding` — укажите тип привязки для Single Sign-on. Большинство поставщиков поддерживают тип привязки `POST`.

- API

    1. [Получите идентификатор каталога](../../../resource-manager/operations/folder/get-id.md), в котором вы будете создавать федерацию.
    1. Создайте файл с телом запроса, например `body.json`:

        ```json
        {
          "folderId": "<ID каталога>",
          "name": "my-federation",
          "autocreateUsers": true,
          "cookieMaxAge":"43200s",
          "issuer": "https://accounts.google.com/o/saml2?idpid=C03xolm0y",
          "ssoUrl": "https://accounts.google.com/o/saml2/idp?idpid=C03xolm0y",
          "ssoBinding": "POST"
        }
        ```

        Где:

        * `folderId` — идентификатор каталога.
        * `name` — имя федерации. Имя должно быть уникальным в каталоге.
        * `autocreateUsers` — флаг, активирующий автоматическое создание новых пользователей в облаке после аутентификации на IdP-сервере. Эта опция упрощает процесс заведения пользователей, но созданному таким образом пользователю по умолчанию назначается только роль `resource-manager.clouds.member`: он не сможет выполнять никаких операций с ресурсами в этом облаке. Исключение — те ресурсы, на которые назначены роли системной группе `allUsers` или `allAuthenticatedUsers`.

            Если эту опцию не включать, то пользователь, которого не добавили в облако, не сможет войти в консоль управления, даже если пройдет аутентификацию на вашем сервере. В этом случае вы можете управлять белым списком пользователей, которым разрешено пользоваться Яндекс.Облаком.
        * `cookieMaxAge` — время, в течении которого браузер не должен требовать у пользователя повторной аутентификации.
        * `issuer` — идентификатор IdP-сервера, на котором должна происходить аутентификация.

            Скопируйте сюда ссылку, указанную в поле **Идентификатор объекта** на странице **Сведения о поставщике услуг идентификации Google** в G-Suite. Это ссылка в формате:

            ```
            https://accounts.google.com/o/saml2?idpid=<ID SAML-приложения>
            ```
        * `ssoUrl` — URL-адрес страницы, на которую браузер должен перенаправить пользователя для аутентификации.

            Скопируйте сюда ссылку, указанную в поле **URL Системы единого входа** на странице **Сведения о поставщике услуг идентификации Google** в G-Suite. Это ссылка в формате:

            ```
            https://accounts.google.com/o/saml2/idp?idpid=<ID SAML-приложения>
            ```
        * `ssoBinding` — укажите тип привязки для Single Sign-on. Большинство поставщиков поддерживают тип привязки `POST`.
    1. {% include [include](../../../_includes/iam/create-federation-curl.md) %}

{% endlist %}

## Укажите сертификаты для федерации {#add-certificate}

{% include [federation-sertificates-note](../../../_includes/iam/federation-sertificates-note.md) %}

Скачайте сертификат с открытой страницы **Сведения о поставщике услуг идентификации Google** в G-Suite. Добавьте этот сертификат в созданную федерацию.

{% include [federation-sertificates-add](../../../_includes/iam/federation-sertificates-add.md) %}

## Получите ссылку для входа в консоль {#get-link}

{% include [federation-login-link](../../../_includes/iam/federation-login-link.md) %}

## Завершите создание SAML-приложения {#configure-sso-gsuite-finish}

Когда создали федерацию и получили получили ссылку для входа в консоль, завершите создание SAML-приложения в G-Suite:
1. Откройте снова окно создания SAML-приложения и нажмите **Далее**.
1. Введите название SAML-приложения, например <q>yandex-cloud-federation</q>. Если надо, добавьте описание и логотип. Нажмите **Далее**.
1. Укажите сведения о Яндекс.Облаке, которое выступает в роли поставщика услуг:
    * В полях **URL ACS** и **Идентификатор объекта** введите полученную ранее [ссылку для входа в консоль](#get-link).
    * Включите опцию **Подписанный ответ**.
    * В поле **Идентификатор названия** укажите, что будет сервер будет возвращать в качестве Name ID — уникального идентификатора пользователя федерации.

        Выберите **Общие сведения**, а рядом — **Основной адрес эл. почты**.
    * Остальные поля необязательные, поэтому можете не использовать их и нажать **Далее**.

    ![image](../../../_assets/iam/federations/configure-saml-gsuite.png)
1. Чтобы пользователь мог обратиться в техподдержку Яндекс.Облака из [консоли управления](https://console.cloud.yandex.ru/support), нажмите **Добавить сопоставления** и настройте, чтобы сервер передавал адрес электронной почты пользователя. Также рекомендуется передавать имя и фамилию пользователя. После этого нажмите **Готово**.

    ![image](../../../_assets/iam/federations/specify-claims-mapping-gsuite.png)
1. На следующей странице вы можете проверить введенные данные вашего SAML-приложения.
1. Включите ваше SAML-приложение, нажав **Изменить статус приложения**.
1. На открывшейся странице выберите, кому будет доступна аутентификация в этой федерации:
    * Чтобы включить доступ для всех пользователей федерации, выберите **Включено для всех**.
    * Чтобы включить доступ для отдельного организационного подразделения, выберите подразделение в списке слева и настройте статус сервиса для этого подразделения. По умолчанию дочерние подразделения наследуют настройки доступа от родительского подразделения.

## Добавьте пользователей в облако {#add-users}

{% include [add-federated-users](../../../_includes/iam/add-federated-users.md) %}

## Протестируйте аутентификацию {#test-auth}

Теперь, когда вы закончили настройку сервера, вы можете протестировать, что все работает:

1. Откройте браузер в гостевом режиме или режиме инкогнито, чтобы не нарушить работу в консоли с аккаунтом на Яндексе.
1. Перейдите по [ссылке для входа в консоль](#get-link), полученной ранее. Браузер должен перенаправить вас на страницу аутентификации в Google.
1. Введите ваши данные для аутентификации. По умолчанию, необходимо ввести UPN и пароль и нажать **Sign in**.
1. После успешной аутентификации, сервер перенаправит вас обратно по по ссылке для входа в консоль, а после на главную страницу консоли управления. В правом верхнем углу вы можете увидеть, что вы вошли в консоль от имени федеративного пользователя.

#### Что дальше {#what-is-next}

* [Назначьте роли добавленным пользователям](../roles/grant.md#access-to-federated-user).