Добавление веб-ресурса
для использования его в шаблоне velocity,
используемом сервлетом.

Привет,
я разрабатываю плагин для Jira service desk
и мне нужно создать форму для настроек выполняемых админом

Я сделовал этому руководству
https://developer.atlassian.com/server/framework/atlassian-sdk/creating-an-admin-configuration-form/?_ga=2.206188571.967677303.1587555306-81712656.1585173891

и у меня есть такой сервлет в моем файле plugin.xml :



<servlet key="admin-servlet" class="org.atlassian.tutorial.actions.AdminServlet">
  <url-pattern>/test/admin</url-pattern>
</servlet>


Класс AdminServlet использует template renderer для возврата
velocity template:

response.setContentType("text/html;charset=utf-8");
renderer.render("/templates/react-panel.vm", response.getWriter());

и перенаправляет пользователя на страницу входа,
если не является администратором
или не вошел в систему.

Template рендерит ReactJS элементы,
которые предоставляет веб-ресурс:

<web-resource key="component-embeddable-pack"> 
  <resource type="download" name="component.pack.js" location="/client/component.pack.js"/>
</web-resource>

Сам шаблон содержит следующее:

<div id="create-article-form">React code should render here!</div>
$webResourceManager.requireResource("org.atlassian.tutorial.react-component-test:component-embeddable-pack")

и этот шаблон правильно отображает компоненты React,
когда является часть web-panel:

<web-panel name="react-panel" i18n-name-key="react--panel.name" key="react--panel" location="atl.jira.view.issue.left.context" weight="50"> 
  <description key="react--panel.description">A react-panel Plugin</description>  
  <resource name="view" type="velocity" location="templates/react-panel.vm"/> 
</web-panel>

которую я создал ранее для целей тестирования.

Итак, вернемся к странице администратора - 
когда я захожу на http://localhost:2990/jira/plugins/servlet/test/admin

шаблон отображатеся,
перенаправление на страницу входа в систему работает отлично,
но компонент React не отображается.
Если я выведу React code в console.log,
то вижу что метод mount компонента вообще не вызывается.
Таким образом, похоже, что шаблон 
не видит веб-ресурс, содержащий код React,
несмотря на то этот ресурс требуется внутри шаблона.
Нужно ли запрашивать веб-ресурс в другом месте 
или я что-то упускаю?


Обновление:
у меня это работает, поэтому в интересах тех,
у кого есть подобные проблемы,
я постараюсь обобщить то,
что я сделал.


Шаблон Velocity выглядит так:

<html>
<head>
    <title>Configuration</title>
    <meta name="decorator" content="atl.admin" />
    <meta name="admin.active.section" content="plugin-admin-config-link" />
</head>
<body>
<h1>Plugin Configuration</h1>
<div id="create-article-form">React code should render here</div>
</body>
</html>


Есть web resourceб который представляет ReactJS bundle:


<web-resource key="component-embeddable-pack"> 
  <resource type="download" name="component.pack.js" location="/client/component.pack.js"/>
  <context>atl.admin</context>
</web-resource>


Для создания пункта меню для администратора у меня есть web-section:

<web-section key="admin_handler_config_section" location="admin_plugins_menu">
  <label key="Plugin - Admin Configuration" />
</web-section>

и web-item которая ссылается на него,
так что элемент меню появляется в этом разделе

<web-item key="plugin-admin-config-link"
          section="admin_plugins_menu/admin_handler_config_section">
  <label key="Plugin Configuration" />
  <link linkId="handler.plugin.configuration.link" key="plugin-configuration">/plugins/servlet/aws/admin</link>
</web-item>

web-item link - это путь к сервлету,
который обслуживает шаблон.

<servlet key="admin-servlet" class="org.atlassian.tutorial.actions.AdminServlet">
  <url-pattern>/aws/admin</url-pattern>
</servlet>

Класс, который реализует сервлет AdminServlet,
использует pageBuilderService
для того чтобы to pull in the component-embeddable-pack web-resource:

pageBuilderService.assembler().resources()
        .requireWebResource("org.atlassian.tutorial.react-component-test:component-embeddable-pack")
            .requireContext("atl.admin");
response.setContentType("text/html;charset=utf-8");
renderer.render("/templates/react-panel.vm", response.getWriter());

Вы объявляете PageBuilderService следующий образом:

@ComponentImport
private PageBuilderService pageBuilderService;

и в конструкторе

this.pageBuilderService = pageBuilderService;

Надеюсь это поможет кому-то.


