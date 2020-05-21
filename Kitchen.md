Из этого учебника вы узнаете,
как написать простой плагин для Atlassian Jira Server.
Урок посвящен Jira 7
и вы сможете пошаго пройти все необходимые этапы.
Обратите внимание что мы разрабатываем плагин только
для Hosted Jira instances only.
Если вы заинтересованы в разработке плагинов для джира клауд,
мы можете попробовать Atlassian Jira Cloud.

Целевая аудитория.
Это руководство подразумевает,
что у вас уже есть знания о разработке программного обеспечения
и вы знаете что такое джава и maven.
Вам не потребуется предварительных знаний о разработке Jira.
Но вы должны также знать, что такое джира
и о том что у нее есть система плагинов
при помощи которой вы можете загружать в джиру внешние плагины 
и затем запускать их.

Какую проблему должен решить плагин?
Вы, вероятно уже используете джиру,
но одна вещь выходит из под контроля:
на кухне часто не хватает планирования.
Почему посудомоечная машина полна грязной посуды, но не включена?
Почему нет чистых блюд?
Это та задача, которую мы хотим решить при помощи нашего плагина.

Желаемые функции как пользовательские истории

Как начальник офиса, я хочу назначать людей на кухню
еженедельно, чтобы улучшить порядок на кухне.

Критерии приемки
Есть возможность указать одного или нескольких пользователей джиры 
для работе на кухне в течении выбранной недели.
Страница на которой устанавливаются настройки
доступна только для администраторов джиры.


Вторая история:
Как главный по офису я хочу видеть,
кто назначен на кухню в течении текущей недели.

Критерии приемки:
Показывать пользователей джиры, у которых на этой недели есть
дежурство на кухне.
Показать весь месяц в календаре,
который можно кликать, чтобы увидеть будущие месяцы.
Страница должна быть доступна для все аутентифицированных пользователей.
Страница должна иметь кнопку для навигации в верхней части страницы
(цель - добавить кнопку: когда у меня есть обязанности на кухне?,
чтобы перейти к месяцу и неделе в календаре,
когда вошедший в систему пользователь джира
назначен для выполнения обязанностей на кухне)


#Начинаем
Установите Atlassian SDK и java 8.
После установки все команды atlas-* и java должны
быть добавлены в переменную PATH

проверьте версию атлассиан SDK

atlas-version
ATLAS Version:    6.1.0
ATLAS Home:       /Applications/Atlassian/atlassian-plugin-sdk-6.1.0
ATLAS Scripts:    /Applications/Atlassian/atlassian-plugin-sdk-6.1.0/bin
ATLAS Maven Home: /Applications/Atlassian/atlassian-plugin-sdk-6.1.0/apache-maven-3.2.1
AMPS Version:     6.1.2
--------
Executing: /Applications/Atlassian/atlassian-plugin-sdk-6.1.0/apache-maven-3.2.1/bin/mvn --version -gs /Applications/Atlassian/atlassian-plugin-sdk-6.1.0/apache-maven-3.2.1/conf/settings.xml
Java HotSpot(TM) 64-Bit Server VM warning: ignoring option MaxPermSize=256M; support was removed in 8.0
Apache Maven 3.2.1 (ea8b2b07643dbb1b84b6d16e1f08391b666bc1e9; 2014-02-14T18:37:52+01:00)
Maven home: /Applications/Atlassian/atlassian-plugin-sdk-6.1.0/apache-maven-3.2.1
Java version: 1.8.0_45, vendor: Oracle Corporation
Java home: /Library/Java/JavaVirtualMachines/jdk1.8.0_45.jdk/Contents/Home/jre
Default locale: de_DE, platform encoding: UTF-8
OS name: "mac os x", version: "10.10.5", arch: "x86_64", family: "mac"


проверьте что джава корректно установлена

java -version
java version "1.8.0_45"
Java(TM) SE Runtime Environment (build 1.8.0_45-b14)
Java HotSpot(TM) 64-Bit Server VM (build 25.45-b02, mixed mode)

## Теперь создайте базовый шаблон проекта

Теперь мы создадим джира-плагин 
при помощи команды atlas-create-jira-plugin
Перейдите в директорию
в которой вы хотите создать свой плагин.
Вам будет предложено указать groupId,
artifactId, version.
Вы можете увидеть мой выбор ниже:


atlas-create-jira-plugin
...
Define value for groupId: : com.comsysto
Define value for artifactId: : kitchen-duty-plugin
Define value for version:  1.0.0-SNAPSHOT: :
Define value for package:  com.comsysto: : com.comsysto.kitchen.duty
....
Confirm
Y
...

Теперь команда должна создать директорию
с именем artifacId
в нашем случае это kitchen-duty-plugin


## Откройте проект в Java IDE

Используйте вашу любиму IDEчтобы открыть проект.
Когда вы откроете проект,
нам нужно создать 3 файла
для лучшего использования git
и для того чтобы сообщить IDE какие мы хотим использовать отступы.

Создайте файл .editorconfig 
и добавьте операторы для конца строки,
кодировки, отступов и табуляции.
Плагин EditorConfig в IntelliJIdea установлен по умолчанию.
Если вы используете другу IDE, то установите плагин.


Содержимое файла .editorconfig

'''
# EditorConfig is awesome: http://EditorConfig.org
root = true
[*]
end_of_line = lf
insert_final_newline = true
charset = utf-8

[*.html]
indent_style = space
indent_size = 4

[*.js]
indent_style = space
indent_size = 4

[*.css]
indent_style = space
indent_size = 4

[*.xml]
indent_style = space
indent_size = 4

[*.java]
indent_style = space
indent_size = 4

[*.vm]
indent_style = space
indent_size = 4
'''

Создайте .gitignore и добавьте правила
игнорирования для файлов, которые мы не хотим коммитить:

содержимое файла .gitignore

'''
.idea/
*.iml
*.log
target/
'''

создайте файл  .gitattributes и установите завершение строк, как в Unix


'''
*.svg eol=lf
*.html eol=lf
*.js eol=lf
*.css eol=lf
*.scss eol=lf
*.json eol=lf
*.sh eol=lf
*.yml eol=lf
*.md eol=lf
*.txt eol=lf
*.java eol=lf
*.properties  eol=lf
*.xml eol=lf
'''

Созданные шаблон проекта
имеет некоторые незначительные недостатки,
и некоторые вещи в нем еще отсутствуют,
и мы должны эти вещи добавить перед тем как сможем выполнить первую сборку нашего проекта.

Отредактируйте pom.xml и добавьте строку с верией плагина
в maven-compiler-plugin



'''
<plugin>
    <artifactId>maven-compiler-plugin</artifactId>
    <version>3.3</version>
    <configuration>
        <source>1.7</source>
        <target>1.7</target>
    </configuration>
</plugin>
'''

Также добавьте в pom.xml в раздел dependencies следующий код:

'''
<dependencies>
   ...
   <dependency>
       <groupId>com.atlassian.sal</groupId>
       <artifactId>sal-api</artifactId>
       <version>3.0.3</version>
       <scope>provided</scope>
   </dependency>
</dependencies>
'''

Теперь откройте настройки в IntelliJ Idea
и установите maven из atlassian sdk,
также settigns.xml из атлассиан sdk,
и репоторий тоже из atlassian sdk.

Совет: при вызове команды atlas-version в консоли 
эта команда печатает все нужные вам пути.

https://comsysto.github.io/kitchen-duty-plugin-for-atlassian-jira/images/doc/step-00-intellij-maven-install-settings.png

Поскольку нам нужна переменная окружения ATLAS_HOME
чтобы мы смогли заставить maven работать внутри IntelliJ IDEA,
то мы также установим следующую настройку:

https://comsysto.github.io/kitchen-duty-plugin-for-atlassian-jira/images/doc/step-00-intellij-maven-atlas-home.png

Теперь вы можете щелкнуть по вкладке Maven вверху справа
и нажать compile 
и проект должен успешно скомпилироваться.

https://comsysto.github.io/kitchen-duty-plugin-for-atlassian-jira/images/doc/step-00-intellij-maven-successful-compile.png

Если вы вызовете в меню Build>Make Project 
прямо сейчас
и откроете MyPluginComponentImpl.java
то у вас не должно быть ошибок.

Вы выполнили шаг 0. Отлично.
Вы можете проверить выполнение здесь
https://github.com/comsysto/kitchen-duty-plugin-for-atlassian-jira/tree/master/step-00-kitchen-duty-plugin

Дальнейшее чтение: OSGI

Вы можете пропустить эту глау, если вы уже знаете OSGi
или вам не интересуют глубокие внутренние аспекты
импорта и экспорта
между модуля OSGi.
Следующий шаг учебника будет основан на этих знаниях.

Открыв MyPluginComponentImpl.java
вы уже могли видеть некоторые аннотации,
такие как @ExportAsService
и @ComponentImport

В файле pom.xml есть также важный раздел maven-jira-plugin
который содержит следующие строки

...
<artifactId>maven-jira-plugin</artifactId>
...
<instructions>
    <Atlassian-Plugin-Key>${atlassian.plugin.key}</Atlassian-Plugin-Key>
    <Export-Package>
       com.comsysto.kitchen.duty.api,
    </Export-Package>
    <Import-Package>
       org.springframework.osgi.*;resolution:="optional",
       org.eclipse.gemini.blueprint.*;resolution:="optional",
       *
   </Import-Package>
   <Spring-Context>*</Spring-Context>
</instructions>

Теперь мы погрузимся в понимание того, что это все значит.
Сначала вы должны прочитать статью в Википедии про OSGi.

Сервисы osgi

каждый бандл может публиковать и искать интерфейсы
и следовательно использовать функциональный возможности других бандлов.
Джира сама состоит из некоторых osgi бандлов.
Поэтому нам нужно сообщить Atlassian SDK,
какие интерфейсы JIRA мы хотим использовать (
среда osgi автоматически найдет для вас нужный бандл)


жизненный цикл osgi

мы можем устанаваливать, запускать и останаваливать бандлы osgi во время выполнения.
Это означает, что во время работы джира наш плагин может быть установлен
запущен или остановлен.


Бандлы osgi

бандлы (также называемые модулями)
только шарят интерфейся для других бандлов.
Вы при этом скрываете свою реализацию.
Atlassian SDK создает файл jar
с настраиваемым файлом MANIFEST.mf
на основании инструкций Import/Export-Package в pom.xml
и аннотаций плагина Atlassian Plugin Scanner,
таких как @ComponentImport


https://comsysto.github.io/kitchen-duty-plugin-for-atlassian-jira/images/drawings/00-mock-osgi-and-jira_big.jpg

мы запускаем команду package для тогочтобы сбилдить jar-файл.


atlas-package
...
[INFO] Building jar: /Users/../kitchen-duty-plugin/target/kitchen-duty-plugin-1.0.0-SNAPSHOT.jar
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------


Теперь мы можем проверить содержимое файла MANIFEST.mf (я его немного укоротил)

cat target/classes/MANIFEST.md
Manifest-Version: 1.0
Atlassian-Build-Date: 2015-12-15T18:43:43+0100
Atlassian-Plugin-Key: com.comsysto.kitchen-duty-plugin
Bundle-Name: kitchen-duty-plugin
Bundle-SymbolicName: com.comsysto.kitchen-duty-plugin
Bundle-Vendor: Comsysto Reply
Bundle-Version: 1.0.0.SNAPSHOT
Export-Package: com.comsysto.kitchen.duty.api;version="1.0.0"
Import-Package: org.springframework.osgi.service.exporter.support;resolution:=optional,
org.springframework.osgi.service.importer.support;resolution:=optional,
....
Require-Capability: osgi.ee;filter:="(&(osgi.ee=JavaSE)(version=1.7))"
Spring-Context: *


Мы видим операторы Export-Package и Import-Package
со значениями, который мы видели в pom.xml
или с теми которые были обозначены аннотациям Atlassian Spring Scanner плагина.



Резюме

@ExportAsService используется для экспорта нашего интерфейса (
например MyPluginComponent.java)
так чтобы другие osgi бандлы могли использовать этот сервис
как Spring Bean.

Может быть, вы хотите разработать базовый плагин
и использовать его функциональность из второго плагина,
тогда теперь вы знаете как теперь это делать.

@ComponentImport используется для импорта функциональности из других osgi бандлов
Spring Context,
таких как JIRA ApplicationProperties.

Эти аннотации используются потому,
что Jira использует Spring
вместе с osgi,
и эти аннотации помогают сделать бины
доступными в виде Spring Beans
и получать из из контекста Spring
(даже если это контекст Spring из другого пакета)

Если мы хотим сделать что-то низкоуровневое с импоротов и экспортом OSGI,
то мы можем сделать это в pom.xml 
при помощи тегов <Import-Package>
и <Export-Package>

Если вы хотите еще глубже погрузиться в osgi и atlassian
прочитайте о плагине Atlassian Spring Scanner
посмотрите видео о Apache Felix
и познакомьтесь с учебниками по Apache Felix.

--------------------------------------------------

Наш базовый проект успешно компилируется,
и теперь мы хотим разобраться с нашей первым сценарием использования:
а именно реализовать страницу "Планирование работы на кухне"

Но прежде чем мы начнем реализацию,
мы выпьем еще один кофе
и доведем наш мозг
до скорости разработки архитектуры программного обеспечения,
потому что нам нужно принять некоторые дизайнерские решения сейчас.
Подумайте о следующей разделе как о небольшом семинаре по истории,
который проводим с нашим вирутальным владельцем продукта
и виртуальной командой скрам,
чтобы сформировать общее понимание предстоящей работы.

Что на самом деле нужно сделать странице планирования?
Нам нужно получить список всех имен
пользователей джиры и сделать их доступными для выбора.
Нам нужен какой-то виджет календаяр,
по которому можно нажимать,
и чтобы там была возможность выбирать недели.
Нам необходимо постоянно сохранять инфомрацию 
в каком-либо хранилище, в базе данных.
Мы должны быть в состоянии получить сохраненных пользователей
для заданной недели,
должны иметь возможность показать их 
и иметь возможность удалить их из графика недель.
Хорошо, это звучит как базовый материал,
который мы делали триллион раз уже в некоторых наших проектах.
Но вопрос в том, как мы можем сделать это классным способом в Джире?

##UX Компоненты (Atlassian AUI)
Большинстово из вас могут знать Bootstrap
и использовать его для создания приятных адаптивных сайтов.
У Атлассиан также есть такая платформа,
которые работает в приложениях атлассиан.
Она называется Atlassian User Interface или кратко AUI

Скриншот интерфейса пользователя Atlassian.

Список покупок.

AUI Select  2- компонент для выбора пользователя.

AUI Select 2 demo.

Компонент AUI Datepicker для выбора недели.

AUI select 2 demo.

Хорошо, теперь мы выбрали два компонента для веб-интерфейса,
которые мы хотим использовать,
и продолжим работу с этими компонентами на стороне сервера.

## Планирование компонентов
Кофеин вступает в силу сейчас,
и наш супер-мозг готов к разработеке серверных компонент.

Мы могли бы использовать сервлет и
просто MVC с JSP-страницами
и обычным циклом запроса-ответа.

Но так как это просто неудачно,
мы сделаем это с REST Resources 
и простым view,
содержащим много (некрасивого)
кода JavaScript
для взаимодействия с REST Resources.

Поэтому, немного подумав, мы придумали схему справа.


============здесь нужно вставить картинку =============
Что такое web action? 
Мы это узнаем немного позже.


## Обзор компонентов
Webwork action - это базовый "servlet"
который предоставляет реальзую страницу для просмотра,
а также обеспечивает представление HTML и JavaScript.

User Search Resource JS Controller содержит набор Js-кода
для взаимодействия между HTML-представлением
и пользовательским ресурсом REST

Duty Planning Resource JS controller содержит набор js кода
для взаимодействия между HTML-представлением
и REST ресурсовм kitcken duty.

HTML View это наш шаблон, который предоставляет базовые
компоненты HTML для подключения контроллера javascript.

User Search REST Resource предоставляет ресурсы REST
для поиска имен пользователей JIRA через JAX-WS.

User Search Resource Model - это наша простая Pojo модель
которую мы используем для хранения данных,
а также в качестве представления json в javascript контроллере.


Kitchen Duty Planning REST Resource предоставляет ресурсы REST для операций CRUD
на KitchenDutyPlanningModel через JAX-WS

Kitchen Duty Planning Model - это наша простая Pojo модельб
которую мы используем для сохранения данных,
а также в качетсве представления JSON в контроллере javascript.


# Webwork Action and HTML View

что вы можете сказать о WebWorkAction?
эта штука появилась из ниоткуда!
И вы правы.
Я познакомлю вас с этим,
так как это один из способов
добавлять страницы внутрь джиры
и это неплохой способ.

Вы можете прочитать о джира jira webwork action
или о webwork plugin module,
но можете это не делать,
так как поймете что происходит
прочитав мое котороткое вступление.

Вступление
Поу сути, веб-действия это контроллеры в парадигме MVC
которые реагируют на определенные URL-адреса в JIRA.

Они легко обрабатывают аутентификацию (пользователь вошел в систему?)
и авторизацию (имеет ли пользователь права на просмотр страницы?)

Они позволяют нам определить несколько представления
согласно парадигме MVC,
например для просмотра ошибки 
или просмотра для успешного состояния.
Они предоставляют Template Renderer,
такой как Velocity,
который помогает нам вводить некоторые переменные в View.

Их легко определить при помощи некоторого XML-кода
в файле-дескрипторе плагина atlassian-plugin.xml (мы это обсудим позже)

Хорошо, теперь вы знаете достаточно, чтобы начать.
Откройте терминал и перейдите в директорию плагина.

Мы снова будем использовать Atlassian SDK
для создания основных файлов.
Введите команду
и выберите 31 (Webwork Plugin)


atlas-create-jira-plugin-module
Choose Plugin Module:
1:  Component Import
2:  Component
3:  Component Tab Panel
...
31: Webwork Plugin
...
34: Workflow Validator
Choose a number (1/2/3.../33/34): 31
Enter Plugin Module Name My Webwork Module: : Kitchen Duty Planning Webwork Module
Show Advanced Setup? (Y/y/N/n) N: : Y
Module Key kitchen-duty-planning-webwork-module: :
Module Description The Kitchen Duty Planning Webwork Module Plugin: :
i18n Name Key kitchen-duty-planning-webwork-module.name: :
i18n Description Key kitchen-duty-planning-webwork-module.description: :
Enter Action Classname MyActionClass: : KitchenDutyPlanningWebworkAction
Enter Package Name com.comsysto.jira.webwork: : com.comsysto.kitchen.duty.webwork
Enter Alias KitchenDutyPlanningWebworkAction: :
Enter View Name success: : kitchen-duty-planning-success
Enter Template Path /templates/.../kitchen-duty-planning-success.vm: : /templates/kitchen-duty-planning-webwork-module/kitchen-duty-planning-success.vm
Add Another View? (Y/y/N/n) N: : N
Add Another Action? (Y/y/N/n) N: : N
[INFO] Adding the following items to the project:
[INFO]   [class: com.comsysto.kitchen.duty.webwork.KitchenDutyPlanningWebworkAction]
[INFO]   [class: ut.com.comsysto.kitchen.duty.webwork.KitchenDutyPlanningWebworkActionTest]
[INFO]   [dependency: org.apache.httpcomponents:httpclient]
[INFO]   [dependency: org.mockito:mockito-all]
[INFO]   [dependency: org.slf4j:slf4j-api]
[INFO]   [module: webwork1]
[INFO]   [resource: kitchen-duty-planning-success.vm]
[INFO]   i18n strings: 2
Add Another Plugin Module? (Y/y/N/n) N: : N
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time: 10:53 min
[INFO] Finished at: 2015-12-16T14:12:07+01:00
[INFO] Final Memory: 29M/309M
[INFO] ------------------------------------------------------------------------


После того как мы это сделали,
мы видим что довольно много кода и файлов было сгенерировано.
Вы можете видеть все изменения в этом Github Commit 162ed70

Для нас важны следующие изменения:

atlassian-plugin.xml теперь содержит определение нашего webwork
для Action и View
для действия и представления.


<webwork1 key="kitchen-duty-planning-webwork-module" name="Kitchen Duty Planning Webwork Module" i18n-name-key="kitchen-duty-planning-webwork-module.name">
    <description key="kitchen-duty-planning-webwork-module.description">The Kitchen Duty Planning Webwork Module Plugin</description>
    <actions>
      <action name="com.comsysto.kitchen.duty.webwork.KitchenDutyPlanningWebworkAction" alias="KitchenDutyPlanningWebworkAction">
        <view name="kitchen-duty-planning-success">/templates/kitchen-duty-planning-webwork-module/kitchen-duty-planning-success.vm</view>
      </action>
    </actions>
  </webwork1>


pom.xml содержит теперь необходимые зависимости.

<dependency>
   <groupId>org.apache.httpcomponents</groupId>
   <artifactId>httpclient</artifactId>
   <version>4.1.1</version>
   <scope>test</scope>
</dependency>
<dependency>
   <groupId>org.slf4j</groupId>
   <artifactId>slf4j-api</artifactId>
   <version>1.6.6</version>
   <scope>provided</scope>
</dependency>
<dependency>
   <groupId>org.mockito</groupId>
   <artifactId>mockito-all</artifactId>
   <version>1.8.5</version>
   <scope>test</scope>
</dependency>

Теперь вы можете открыть KitchenDutyPlanningWebworkAction в IntellijIdea.

https://comsysto.github.io/kitchen-duty-plugin-for-atlassian-jira/images/doc/step-01-intellij-after-webwork-module-create.png

Теперь мы можем изменить метод execute для работы с нашим представлением.


src/main/java/com/comsysto/kitchen/duty/webwork/KitchenDutyPlanningWebworkAction.java


@Override
public String execute() throws Exception {
   return "kitchen-duty-planning-success";
}


Теперь наше представление будет вызвано,
когда мы перейдем по URL адресу веб-действий web-actions url,
http://server/jira/secure/[WebWorkAlias].jspa
который в нашем случае будет просто http://server/jira/secure/KitchenDutyPlanningWebworkAction.jspa.


Теперь мы запустим JIRA, 
в которой будет установлен наш плагин
с помощью следующей команды

atlas-run
...
[INFO] [talledLocalContainer] INFORMATION: Server startup in 37754 ms
[INFO] [talledLocalContainer] Tomcat 8.x started on port [2990]
[INFO] jira started successfully in 59s at http://localhost:2990/jira
[INFO] Type Ctrl-D to shutdown gracefully
[INFO] Type Ctrl-C to exit

(теперь вы можете сходить еще за кофе,
потому что установка всех зависимостей займет некоторое время)

Теперь перейдите по адресу
http://localhost:2990/jira/secure/KitchenDutyPlanningWebworkAction.jspa 
и вы должны увидеть:

https://comsysto.github.io/kitchen-duty-plugin-for-atlassian-jira/images/doc/step-01-atlas-run-jira-with-kitchen-duty-planning-webwork-action.png

Хорошо, Это работате, но выглядит не очень хорошо.
Мы будем улучшать внешний вид и функциональность шаг за шагом.

что нам нужно для улучшения UX?

Мы хотим иметь обычный макет страницы администратора JIRA.
Нам нужна боковая панель навигации для ссылок на наши страницы
Конечно, нам нужен раздел контента
i18n было бы неплохо, чтобы потом мы могли легко сделать наш плагин многоязычным.
Страница планировать должна быть доступна только администраторам.

## JIRA administrator page layout (Page Decorator)
Мы знаем что нам нужно,
и я познакомлю вас с компонентами Atlassian,
которые выполнят эту работу.

Отредактируйте файл kitchen-duty-planning-cussess.vm
который находится в src/main/resources/templates/kitchen-duty-planning-webwork-module
так как показано ниже:

src/main/resources/templates/kitchen-duty-planning-webwork-module/kitchen-duty-planning-success.vm



<html>
<head>
    <title>Planning Page - Kitchen Duty Plugin</title>
    <meta name="decorator" content="atl.admin">
</head>
<body>
<h1>Kitchen Duty Plugin - Planning Page</h1>

<p>Now it looks nice :)</p>
</body>
</html>


Важной вещью является часть Page Decorator,
в которой написано 
<meta name="decorator" content="atl.admin">

Так о чем это?
Это утверждение говорит декоратору страницы
визуализировать макет для atl.admin
который в основном является макетов для администратора.

После сохранения изменений запустите atlas-run
снова
чтобы стартовать джиру снова
и перейти на страницу Planning Page.
Теперь это должно выглядеть так.

https://comsysto.github.io/kitchen-duty-plugin-for-atlassian-jira/images/doc/step-01-planning-page-decorator-atl-admin.png

Если у вас еще работает atlas-run
и вы не видите изменений,
попробуйте открыть вторую оболочку shell
и введите atlas-cli,
дождите приглашения командной строки,
введите pi
и нажмите Enter.
Теперь ваш плагин принудительно пересобран.
(это уже не работает кажется?)

Вы можете прочитать об этом в разделе о быстрой разработкею
faster development.

2 Sidebar Navigation (Web section / Item)
Боковая навигация

Мы могли бы просто возиться с нашей собственной боковой панелью,
но это не профессиональный способ.
Сделайте это профессионально,
так как мы будем использовать Web Section Plugin Module
и Web Item Plugin Module,
чтобы подключить к боковой панели Jira,
которая отображается для каждого раздела вкладки администратора.
Мы сделаем это так, чтобы это выглядело следующим образом:

https://comsysto.github.io/kitchen-duty-plugin-for-atlassian-jira/images/doc/step-01-planning-page-web-section-and-item.png


Вам может понадобиться еще один кофе,
потмоу что здесь идет еще один код для
atlassian-plugin.xml,
который находится в src/main/resources/



src/main/resources/atlassian-plugin.xml


<atlassian-plugin>
...
  <web-section key="admin_kitchen_duty_planning_section"
               name="admin_kitchen_duty_planning_section"
               location="admin_plugins_menu"
               weight="20"
               i18n-name-key="kitchen-duty-plugin.admin.planning.page.web.section.name">
    <label key="kitchen-duty-plugin.admin.planning.page.web.section.name" />
  </web-section>

  <web-item key="admin_kitchen_duty_planning_webitem"
            name="admin_kitchen_duty_planning_webitem"
            section="admin_plugins_menu/admin_kitchen_duty_planning_section"
            weight="15"
            i18n-name-key="kitchen-duty-plugin.admin.planning.page.web.item.name">
    <label key="kitchen-duty-plugin.admin.planning.page.web.item.name" />
    <link linkId="admin_kitchen_duty_planning_webitem_link">/secure/KitchenDutyPlanningWebworkAction.jspa</link>
  </web-item>

</atlassian-plugin>


Web-section это "контейнер", который содержит веб-элементы
и имеет заголовок.
Веб-элементы (web-items) это в основном навигационные ссылки,
которые имеют имя и url.



Так что же здесь происходит?
web-section нужен уникальный ключ и имя.
weight определяет порядок раздела относительно разделов других плагинов.
location определяет стараницу или вкладку,
на которой должен быть размещен web-section.
Мы используем вкладку для плагинов для размещения наших web-sections.
Я думаю что label key это что-то из osgi,
просто назовите этот ключ как i18n-name-key.
Наконец, i18n-name-key это идентификатор языкового файла kitchen-duty-plugin.properties
который содержит фактические переводы для текстовых строк.

Web-item имеет те же атрибуты что и web-section.
Но вам нужно указать section 
которые в свою очередь указывает на название вашего web-section (name).
Объект link содержит фактический url нашей страницы.

Поскольку мы узнали о i18n, то нам нужно определить наши
значения в языковом файле kitchen-duty-plugin.properties

src/main/resources/kitchen-duty-plugin.properties


# websection/webitems
kitchen-duty-plugin.admin.planning.page.web.section.name = Kitchen Duty Plugin
kitchen-duty-plugin.admin.planning.page.web.item.name = Planning Page


Хорошо, это немного сложно, но теперь у нас есть хорошая навигационная ссылка на боковой панели.

## 3. Content Section (AUI Layout)
## Раздел содержимого (макет AUI)

Хорошо, это несложно, так как вы уже видели раздел контента,
Но чтобы быть последовательными мы разместим здесь для вас скриншот:

https://comsysto.github.io/kitchen-duty-plugin-for-atlassian-jira/images/doc/step-01-planning-page-content.png


Если вы хотите стать студентом,
вы можете прочитать об AUI Layout
и немного поиграть с ними.
Мы будем использовать это позже для оформления наших страниц.

## 4. Internationalization (i18n / Velocity)

Вы уже видели i18n для названий веб-разделов и элементов (web-sections/web-items)

Теперь мы хотим добавить интернационализацию в наш шаблон HTML.

Возможно, вы уже видели окончание файла .vm.
Это означает, что kitchen-duty-planning-success.vm
является шаблоном Apache Velocity.
Некоторые могут задаться вопросом: Разве древние греки не использовали velocity?
Да, я думаю использовали, и это было очень полезно.

Итак, приступим к работе,
мы изменим наш html-код,
используя i18n.getText("i18n.key")
который является хелпером велосити для рендеринга значений i18n для заданного ключа.


src/main/resources/templates/kitchen-duty-planning-webwork-module/kitchen-duty-planning-success.vm


<html>
<head>
    <title>$i18n.getText("kitchen-duty-plugin.admin.planning.page.title")</title>
    <meta name="decorator" content="atl.admin">
</head>
<body>
<h1>$i18n.getText("kitchen-duty-plugin.admin.planning.page.headline")</h1>

<p>Now it looks nice :)</p>
</body>
</html>


Теперь нам нужно добавить переводы в файл kitchen-duty-plugin.properties,
и таким образом мы завершим интернационализацию нашей страницы.



src/main/resources/kitchen-duty-plugin.properties


# kitchen-duty-planning-success.vm
kitchen-duty-plugin.admin.planning.page.headline = Kitchen Duty Plugin - Planning Page
kitchen-duty-plugin.admin.planning.page.title = Planning Page - Kitchen Duty Plugin


5. Admin only Authorization (Webwork Action)

Мы используем webwork action на нашей странице планирования
и мы можем просто установить role-required="admin"
в atlassian-plugin.xml
чтобы обеспечить авторизацию только для администраторов системы.

src/main/resources/atlassian-plugin.xml

<webwork1 key="kitchen-duty-planning-webwork-module"
          ...
          roles-required="admin">


Если вы вйдете из Jira сейчас и снова перейдете на страницу планирования,
то вы увидите форму входа администратора.
При входе в систему как администратор (введя пароль администратора),
вы должны быть перенаправлены на страницу планирования.



https://comsysto.github.io/kitchen-duty-plugin-for-atlassian-jira/images/doc/step-01-done.png




Вы прошли Шаг 1. Хорошая Работа! Вы можете проверить решение этого шага на GitHub
https://github.com/comsysto/kitchen-duty-plugin-for-atlassian-jira/tree/master/step-01-kitchen-duty-plugin


На рисунке зеленым цветом показаны компоненты, который мы реализовали в этом разделе.


-=------------------------рисунок

#User Search REST Resource

Теперь мы создадим REST ресурс для поиска пользователя
чтобы наш js-контейнер мог искать имена пользователей.

Что такое rest-ресурс?
ЭТо просто базовый контроллер (MVC)
который реагирует на определенные запросы HTTP
определенным ответом.

Atlassian использует JAX-WS и JAXB для встроенных рест-ресурсов.
Фактически реализация скрыта от нас,
и нас это не волнует,
поскольку нам просто нужно знать,
что где-то внутри джира находятся бандлы osgi,
которые обеспечивают реализацию JAX-WS и JAXB.

Давайте создадим наш рест ресурс с помощью команды Atlassian SDK,
подобно тому как мы создавали webwork action ранее

Выполните комнду и выберите пункт 14: Rest Plugin Module
и заполните ответы на вопросы, так как показано ниже:

atlas-create-jira-plugin-module
Executing: /Applications/Atlassian/atlassian-plugin-sdk-6.1.0/apache-maven-3.2.1/bin/mvn com.atlassian.maven.plugins:maven-jira-plugin:6.1.2:create-plugin-module -gs /Applications/Atlassian/atlassian-plugin-sdk-6.1.0/apache-maven-3.2.1/conf/settings.xml
Java HotSpot(TM) 64-Bit Server VM warning: ignoring option MaxPermSize=256M; support was removed in 8.0
[INFO] Scanning for projects...
[INFO]
[INFO] Using the builder org.apache.maven.lifecycle.internal.builder.singlethreaded.SingleThreadedBuilder with a thread count of 1
[INFO]
[INFO] ------------------------------------------------------------------------
[INFO] Building kitchen-duty-plugin 1.0.0-SNAPSHOT
[INFO] ------------------------------------------------------------------------
[INFO]
[INFO] --- maven-jira-plugin:6.1.2:create-plugin-module (default-cli) @ kitchen-duty-plugin ---
Choose Plugin Module:
1:  Component Import
...
14: REST Plugin Module
15: RPC Endpoint Plugin
...
Choose a number (...): 14

Enter New Classname MyRestResource: : UserSearchResource
Enter Package Name com.comsysto.rest: : com.comsysto.kitchen.duty.rest
Enter REST Path /usersearchresource: : /kitchenduty
Enter Version 1.0: : 1.0

Show Advanced Setup? (Y/y/N/n) N: : y

Module Name User Search Resource: : Kitchen Duty Resources
Module Key user-search-resource: : kitchen-duty-resources
Module Description The User Search Resource Plugin: : All Kitchen Duty REST Resources
i18n Name Key user-search-resource.name: : kitchen-duty-plugin.rest.resources.name
i18n Description Key user-search-resource.description: : kitchen-duty-plugin.rest.resources.description
Add Package To Scan? (Y/y/N/n) N: : y
Enter Package: com.comsysto.kitchen.duty.rest
...

Add Package To Scan? (Y/y/N/n) N: : n

Add Dispatcher? (Y/y/N/n) N: : n

[INFO] Adding the following items to the project:
[INFO]   [class: com.comsysto.kitchen.duty.rest.UserSearchResourceModel]
[INFO]   [class: com.comsysto.kitchen.duty.rest.UserSearchResource]
[INFO]   [class: it.com.comsysto.kitchen.duty.rest.UserSearchResourceFuncTest]
[INFO]   [class: ut.com.comsysto.kitchen.duty.rest.UserSearchResourceTest]
[INFO]   [dependency: com.atlassian.plugins.rest:atlassian-rest-common]
[INFO]   [dependency: com.atlassian.sal:sal-api]
[INFO]   [dependency: javax.servlet:servlet-api]
[INFO]   [dependency: javax.ws.rs:jsr311-api]
[INFO]   [dependency: javax.xml.bind:jaxb-api]
[INFO]   [dependency: org.apache.wink:wink-client]
[INFO]   [dependency: org.mockito:mockito-all]
[INFO]   [module: rest]
[INFO]   i18n strings: 2

Add Another Plugin Module? (Y/y/N/n) N: : n

[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time: 01:52 min
[INFO] Finished at: 2016-01-16T20:07:35+01:00
[INFO] Final Memory: 24M/328M
[INFO] ------------------------------------------------------------------------

Atlassin SDK создал и обновил много файлов сейчас,
и прежде чем мы углубимся в детали,
давайте снова запустим atlas-run
и посмотрим что было создано для нас.

После запуска JIRA перейдите по адресу

http://server/jira/rest/kitchenduty/1.0/message

и вы увидите следующее

https://comsysto.github.io/kitchen-duty-plugin-for-atlassian-jira/images/doc/step-02-rest-user-search-response-xml.png


Это мило.
Но теперь нам нужно понять, что происходит.
Итак вот краткое резюме,
того что было создано.

Файл pom.xml был изменен, чтобы обеспечить необходимые зависимости
для API JAX-WS, Servlet API, и также были добавлены некоторые зависимости для тестирования.

Файл atlassian-plugin.xml был изменен для регистрации рест-ресурса
который будет слушать следующий url

http://server/jira/rest/kitchenduty/1.0/*

Теперь есть UserSearchResource.java,
который является фактическим контроллером REST,
этот контроллер и предоставляет ответ.

В дополнение к контроллеру REST 
существует UserSearchResourceModel.java
этот класс представляет собой модель,
которая поставляется в виде XML в ответе рест-контроллера.

Был также создан файл юнит-теста UserSearchResourceTest
и интеграционный тест UserSearchResourceFuncTest

Теперь мы рассмотрим все делати и реализуем необходимые функции
для поиска пользователей на лету.

## 1. Dependencies (pom.xml)

В pom.xml есть новые строки которые говорят сами за себя.
Если вы хотите узнать что они значат подробно, то прочтите документацию к REST Plugin Module

pom.xml


<dependency>
    <groupId>javax.servlet</groupId>
    <artifactId>servlet-api</artifactId>
    <version>2.4</version>
    <scope>provided</scope>
</dependency>
<dependency>
    <groupId>javax.xml.bind</groupId>
    <artifactId>jaxb-api</artifactId>
    <version>2.1</version>
    <scope>provided</scope>
</dependency>
<dependency>
    <groupId>com.atlassian.plugins.rest</groupId>
    <artifactId>atlassian-rest-common</artifactId>
    <version>1.0.2</version>
    <scope>provided</scope>
</dependency>
<dependency>
    <groupId>org.apache.wink</groupId>
    <artifactId>wink-client</artifactId>
    <version>1.1.3-incubating</version>
    <scope>test</scope>
</dependency>


## 2. REST Resources Handler (atlassian-plugin.xml)

Как уже упомниало, файл atlassian-plugin.xml был изменен для регистрации REST ресурса
который должне слушать url
http://server/jira/rest/kitchenduty/1.0/*.

path и версия формируют базовую ссылку относительно http://server/jira/rest/*.

Пакет будет с включенным Component Scan (которое я считаю излишним, но оно не помешает).

src/main/resources/atlassian-plugin.xml



<rest name="Kitchen Duty Resources"
      i18n-name-key="kitchen-duty-plugin.rest.resources.name"
      key="kitchen-duty-resources"
      path="/kitchenduty"
      version="1.0">
  <description key="kitchen-duty-plugin.rest.resources.description">All Kitchen Duty REST Resources</description>
  <package>com.comsysto.kitchen.duty.rest</package>
</rest>



Вы уже знаете i18n-name-key и другие атрибуты из webwork action
и следовательно не удивляетесь, что kitchen-duty-plugin.properties
содержит несколько новых пар ключ-значение



src/main/resources/kitchen-duty-plugin.properties


kitchen-duty-plugin.rest.resources.name=Kitchen Duty Resources
kitchen-duty-plugin.rest.resources.description=All Kitchen Duty REST Resources


## 3. REST Controller and Model

В каталоге src/main/java/
в пакете com.comsysto.kitchen.duty.rest 
теперь есть UserSearchResource.java
и UserSearchResourceModel.java
Вы можете увидеть, что было созданно в соответствующем коммите в гитхаб: 

...

Мы изменим код, чтобы обеспечить работу GET точки входа для поиска имен пользователей.



src/main/java/com/comsysto/kitchen/duty/rest/UserSearchResource.java


package com.comsysto.kitchen.duty.rest;

import com.atlassian.jira.bc.user.search.UserSearchParams;
import com.atlassian.jira.bc.user.search.UserSearchService;
import com.atlassian.plugins.rest.common.security.AnonymousAllowed;

import javax.inject.Inject;
import javax.inject.Named;
import javax.servlet.http.HttpServletRequest;
import javax.ws.rs.GET;
import javax.ws.rs.Path;
import javax.ws.rs.Produces;
import javax.ws.rs.QueryParam;
import javax.ws.rs.core.Context;
import javax.ws.rs.core.MediaType;
import javax.ws.rs.core.Response;
import java.util.ArrayList;
import java.util.List;

@Named 
@Path("/user")  
public class UserSearchResource {

    private UserSearchService userSearchService; 

    @Inject 
    public UserSearchResource(UserSearchService userSearchService) {
        this.userSearchService = userSearchService;
    }

    
    public UserSearchResource() {
    }

    @GET
    @Path("/health") 
    @Produces({MediaType.APPLICATION_JSON})
    @AnonymousAllowed 
    public Response health() {
        return Response.ok("ok").build();
    }

    /**
     * Call from select2 JS plugin
     * Response needs to look like this:
     * [{ 'id': 1, 'text': 'Demo' }, { 'id': 2, 'text': 'Demo 2'}]
     */
    @GET
    @Path("/search") 
    @Produces({MediaType.APPLICATION_JSON})
    public Response searchUsers(@QueryParam("query") final String userQuery, 
                                @Context HttpServletRequest request ) {
        List<UserSearchResourceModel> users = findUsers(userQuery);
        return Response.ok(users).build(); 
    }

    private List<UserSearchResourceModel> findUsers(String query) {
        List<UserSearchResourceModel> userSearchResourceModels =
                                           new ArrayList<UserSearchResourceModel>();
        UserSearchParams searchParams = UserSearchParams.builder()
            .includeActive(true)
            .sorted(true)
            .build();
        List<String> users = userSearchService.findUserNames(query, searchParams); 
        if (users != null) {
            for (String user : users) {
                userSearchResourceModels.add(new UserSearchResourceModel(user, user));
            }
        }
        return userSearchResourceModels;
    }
}




Наш рест контроллер дожне быть Spring-Bean компонентом
и получать зависимости через autowired.
Поэтому установите аннотацию @Named на уровне класса.

Мы хотим, чтобы все конечные точки рест, имели url префикс /user
и поэтому мы используем аннотацию @Path

UserSearchService - это сервис внутри джиры,
который предоставляет нам методы для поиска пользователей.
Этот сервис должен быть импортирован из foreign osgi бандла
об этом мы поговорим позже.
Почему бы не использовать @ComponentImport в данной ситуации, спросите вы?
Хорошо, попробуйте,
и если это работает для вас то окей.
Но я получаю некоторые странные исключения
от WADLGenerator при запуске команды
atlas-integration-test

Поэтому я использую небольшой трюк и испортирую компоненту куда-то еще???

Как только компонент импортирован где-то внутри пакета,
мы можем использовать его и здесь.
Мы увидим это позже.

Мы используе @Inject для инджектирования конструктора,
так что наш UserSearchService будуте внедрен спрингом
при создании бина.

Мы также предоставляем конструктор по умолчанию на всякий случай.

Поскольку наша конечная точка для поиска пользователей
будет довольно сложной,
то мы хотим, чтобы был какой-то health endpoint 
для легкого тестирования API.
Поэтмоу мы определяем метод как @GET
с @Path "/health"
который создает json 
@Produces({MediaType.APPLICATION_JSON})
Эта конечная точка будет доступна по адресу 
http://localhost:2990/jira/rest/kitchenduty/1.0/user/health.

Эта конечная точка будет доступна без аутентификации
мы определяем это при помощи аннотации @AnonymousAllowed

Таким образмо, мы можете легко отправить curl на url и получить какой-то ответ.
Хорошо, теперь мы кодируем конечну точку для поиска
пользователя, путем указания аннотации @Path("/search")

Мы не используем @AnonymousAllowed здесь,
потому что не хотим,
чтобы неаутентифицированные пользователи могли искать в нашей базе данных пользователей.

Поскольку мы хотим искать имя пользователя,
каким-то образмо наше ключевое слово для поиска должны быть передано
как url параметр.
Это делается через @QueryParam("query")

что приводит к следующему url для нашей конечной точки
http://localhost:2990/jira/rest/kitchenduty/1.0/user/search?query=foo.


почему QueryParam вместо PathParam?
Я вижу что вам нравятся красивые url,
и мне тоже,
но позже мы увидим, что виджет AUI select2
будет отправлять ключевое слово для поиска
в качестве параметра запроса,
поэтому мы будем придерживаться поведения по умолчанию.
С помощью @Context HttpServletRequest мы получаем HTTP-запрос
в качестве параметра
и может например извлечь из запроса аутентифицированного пользователя.
Нам это не нужно сейчас, но может в ближайшее время пригодиться.

@Context гарантирует что контекст джиры
(я думаю специальные headers и прочее)
связаны с запросом.

После того, как мы нашли пользователей.
мы строим ответ при помощи Response.ok(someObject).build()

Реализация JAXB/JAX-WS гарантирует,
что наша Модель (мы увидим это позже)
будет преобразована в json
и что http код 200
будет отправлен вместе с представлением нашего объекта
в качестве тела ответа.

Мы предоставляем частный методы, который ищет имена пользователей,
используя UserSearchService.
Если импорт компонентов osgi работате неправильно,
вы получите здесь исключение NullPointerException.
После поиска имен пользователей
мы преобразуем список имен пользователей в нашу модель
UserSerachResourceModel,
которая будет сериализована как тело ответа JSON.




Теперь у нас есть REST Controller,
но теперь у вас будут некоторые ошибки компиляции,
потому что модель должна быть реализована.



src/main/java/com/comsysto/kitchen/duty/rest/UserSearchResourceModel.java


package com.comsysto.kitchen.duty.rest;

import javax.xml.bind.annotation.*;
@XmlRootElement(name = "users") 
@XmlAccessorType(XmlAccessType.FIELD) 
public class UserSearchResourceModel {

    @XmlElement 
    private String text;

    @XmlElement 
    private String id;

    public UserSearchResourceModel() {
    }

    public UserSearchResourceModel(String text, String id) {
        this.text = text;
        this.id = id;
    }

    public String getText() {
        return text;
    }

    public void setText(String text) {
        this.text = text;
    }

    public String getId() {
        return id;
    }

    public void setId(String id) {
        this.id = id;
    }
}

в настоящий момент мы используем только json
и отключили xml
в качестве формата ответа.
Но если вы когда-нибудь захотите использовать xml
вам понадобиться имя для контейнера, содержащего пользователей.
Компилятор ищет аннотацию @XmlElement на уровне атрибутов
а не на методах set/get
Поскольку виджет select2 из AUI хочет иметь определенный формат json,
то мы просто придерживаемся этого и определяем свойств с именем text.
Также мы определяем свойство с именем id.
Хорошо, после этого мы будем получать json следующего формата:

[ { "id": "foo", "text": "foo" }, { "id": "bar", "text": "bar" } ]



Хорошо, теперть мы можем запустить atlas-run
и посмотреть что у нас получится
но как я уже говорил
сначала на мнужно позаботиться о @ComponentImport в UserSearchService
Поскольку не имеет значения, куда в нашем плагине мы импортируем внешние интерфейсы,
то мы сделаем это в "фиктивном" компоненте,
после этого наш импорт будет доступен во всем плагине.

src/main/java/com/comsysto/kitchen/duty/impl/MyPluginComponentImpl.java

package com.comsysto.kitchen.duty.rest;

import com.atlassian.jira.bc.user.search.UserSearchService;
import com.atlassian.plugin.spring.scanner.annotation.imports.ComponentImport;
...

@Named ("myPluginComponent")
public class MyPluginComponentImpl implements MyPluginComponent {
   ...
   @ComponentImport
   private UserSearchService userSearchService;
   ...
}


## 4. Unit- and Integrationtests

Теперь, когда у нас есть готовый код,
нам нужно позаботиться о модульных и интеграционных тестах.

Сначала мы изменим Unitest который находится в
/src/test/java/*
на что-то имеющее смысл.
Таким образом мы мокаем UserSearchService
и предоставляем двух пользователей bob и sue - как мок.

Теперь мы провермяем, работает ли наш метод seachUser как ожидалось.

src/test/java/ut/com/comsysto/kitchen/duty/rest/UserSearchResourceTest.java


package ut.com.comsysto.kitchen.duty.rest;

import com.atlassian.jira.bc.user.search.UserSearchParams;
import com.atlassian.jira.bc.user.search.UserSearchService;
import com.comsysto.kitchen.duty.rest.UserSearchResource;
import com.comsysto.kitchen.duty.rest.UserSearchResourceModel;
import org.junit.After;
import org.junit.Before;
import org.junit.Test;
import org.mockito.Mockito;

import javax.ws.rs.core.Response;
import java.util.ArrayList;
import java.util.List;

import static org.junit.Assert.assertEquals;
import static org.junit.Assert.assertNotNull;
import static org.mockito.Matchers.any;
import static org.mockito.Matchers.anyObject;
import static org.mockito.Matchers.anyString;
import static org.mockito.Mockito.mock;
import static org.mockito.Mockito.when;

public class UserSearchResourceTest {

    @Before
    public void setup() {

    }

    @After
    public void tearDown() {

    }

    @Test
    public void messageIsValid() {
        final List<String> mockedUsers = new ArrayList<>();
        mockedUsers.add("bob");
        mockedUsers.add("sue");

        UserSearchService mockedUserSearchService = Mockito.mock(UserSearchService.class);
        when(mockedUserSearchService.findUserNames(anyString(), any(UserSearchParams.class))).thenReturn(mockedUsers);

        UserSearchResource resource = new UserSearchResource(mockedUserSearchService);

        Response response = resource.searchUsers("bo", null);
        final List<UserSearchResourceModel> users = (List<UserSearchResourceModel>) response.getEntity();

        assertEquals("should contain bob", "bob", users.get(0).getText());
    }
}



теперь, мы можем запустить atlas-unit-test,
эта команда должна выполнить запуск всех ваших юнит-тестов.

atlas-unit-test
[INFO] ------------------------------------------------------------------------
[INFO] Building kitchen-duty-plugin 1.0.0-SNAPSHOT
[INFO] ------------------------------------------------------------------------
[INFO]
[INFO] --- maven-jira-plugin:6.1.2:compress-resources (default-compress-resources) @ kitchen-duty-plugin ---
[INFO] Compiling javascript using YUI
...

-------------------------------------------------------
 T E S T S
-------------------------------------------------------
Running ut.com.comsysto.kitchen.duty.MyComponentUnitTest
Tests run: 1, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 0.051 sec
Running ut.com.comsysto.kitchen.duty.rest.UserSearchResourceTest
Tests run: 1, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 0.148 sec
Running ut.com.comsysto.kitchen.duty.webwork.KitchenDutyPlanningWebworkActionTest
Tests run: 1, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 0 sec

Results :

Tests run: 3, Failures: 0, Errors: 0, Skipped: 0

[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time: 3.477 s
[INFO] Finished at: 2016-03-11T19:34:06+01:00
[INFO] Final Memory: 25M/320M
[INFO] ------------------------------------------------------------------------


Теперь когда юнит-тесты работают, давайте изменим созданные ранее интеграционные тесты.


src/test/java/it/com/comsysto/kitchen/duty/rest/UserSearchResourceFuncTest.java

package it.com.comsysto.kitchen.duty.rest;

import org.apache.wink.client.ClientConfig;
import org.apache.wink.client.Resource;
import org.apache.wink.client.RestClient;
import org.apache.wink.client.handlers.BasicAuthSecurityHandler;
import org.junit.After;
import org.junit.Before;
import org.junit.Test;

import static org.junit.Assert.assertEquals;
import static org.junit.Assert.assertNotNull;

public class UserSearchResourceFuncTest {

    private String baseUrl;

    @Before
    public void setup() {
        baseUrl = System.getProperty("baseurl"); 
    }

    @After
    public void tearDown() {

    }

    @Test
    public void testSearchUser__unAuthorized() { 
        String resourceUrl = baseUrl + "/rest/kitchenduty/1.0/user/search?query=adm";
        String response = httpGet(resourceUrl);
        assertNotNull("should not be null", response);
        assertEquals("should contain unauthorized message",
            toJSON("{'message':'Client must be authenticated to access this resource.','status-code':401}"),
            response);
    }
    @Test
    public void testSearchUser__authorized() { 
        String resourceUrl = baseUrl + "/rest/kitchenduty/1.0/user/search?query=adm";
        String response = httpGet(resourceUrl, "admin", "admin");
        assertNotNull("should not be null", response);
        assertEquals("should contain admin",
            toJSON("[{'text':'admin','id':'admin'}]"),
            response);
    }

    private String toJSON(String text) { 
        return text.replaceAll("'", "\"");
    }

    private String httpGet(String url) { 
        return _httpGet(url, null);
    }

    private String httpGet(String url, String username, String password) { 
        ClientConfig config = new ClientConfig();
        BasicAuthSecurityHandler basicAuthSecHandler = new BasicAuthSecurityHandler();
        basicAuthSecHandler.setUserName(username);
        basicAuthSecHandler.setPassword(password);
        config.handlers(basicAuthSecHandler);
        return _httpGet(url, config);
    }

    private String _httpGet(String url, ClientConfig config) { 
        RestClient client = new RestClient();
        if (config != null) {
            client = new RestClient(config);
        }
        Resource resource = client.resource(url);
        return resource
            .header("Accept", "application/json;q=1.0")
            .get()
            .getEntity(String.class);
    }
}



посколькум мы будем запускать специальную команду SDK
для выполнения наших интеграционных тестов,
SDK system property
с базовым Url адресом для джиры.
Мы установили это на этапе @Before перед тестом.
Наш первый тест должен проверить аутентификацию,
поэтому мы вызваем наш ресурс REST без учетных данных 
и ожидаем получить код ошибки 401ю

Наш второй тест должен проверить валиден ли ответ,
для аутентифицированного пользователя admin.
Мы имем adm и ожидае получить admin как результат.
Это всего лишь небольшой вспомогательный метод
для замены одинарных кавычек на двойные,
так что мы можем легко написать json с одинарными кавычками в наших тестах.
Удобный метод для запуска HTTP GET запроса
с аутентифицаией (базова аутентификация)ю

Низкоуровневый метод HTTP GET,
использующий wink ?? RestClient.
Вам скорее всего нужно будет написать больше,
если ваши тесты станут более сложными,
но для нас этого пока достаточно.

Теперь мы можем запустить atlas-integration-test команду
чтобы запустить наши интеграционные тесты.
Возможно, что вы захотите
выпить еще один кофе, пока будет идти запуск и выполнение тестов,
так как это займет некоторое время.


atlas-integration-test
[INFO] ------------------------------------------------------------------------
[INFO] Building kitchen-duty-plugin 1.0.0-SNAPSHOT
[INFO] ------------------------------------------------------------------------
[INFO]
[INFO] --- maven-jira-plugin:6.1.2:compress-resources (default-compress-resources) @ kitchen-duty-plugin ---
[INFO] Compiling javascript using YUI
[INFO] 0 Javascript file(s) were minified into target directory /Users/bg/git-work/kitchen-duty-MASTER/step-02-kitchen-duty-plugin/target/classes
[INFO] 0 CSS file(s) were minified into target directory /Users/bg/git-work/kitchen-duty-MASTER/step-02-kitchen-duty-plugin/target/classes
[INFO] Compressing XML files
[INFO] 0 XML file(s) were minified into target directory /Users/bg/git-work/kitchen-duty-MASTER/step-02-kitchen-duty-plugin/target/classes
[INFO]
....
[INFO] --- atlassian-spring-scanner-maven-plugin:1.2.6:atlassian-spring-scanner (default) @ kitchen-duty-plugin ---
[INFO] Starting Atlassian Spring Byte Code Scanner...
[INFO]

... 4 million lines later ...

-------------------------------------------------------
 T E S T S
-------------------------------------------------------
Running it.com.comsysto.kitchen.duty.MyComponentWiredTest
Tests run: 1, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 1.248 sec
Running it.com.comsysto.kitchen.duty.rest.UserSearchResourceFuncTest
Tests run: 2, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 1.018 sec

Results :

Tests run: 3, Failures: 0, Errors: 0, Skipped: 0

[INFO] jira: Shutting down
[INFO] using codehaus cargo v1.4.7
...
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time: 01:10 min
[INFO] Finished at: 2016-03-11T19:38:26+01:00
[INFO] Final Memory: 32M/437M
[INFO] ------------------------------------------------------------------------


# Finally let's do a manual test
Окончательный тест выполняем вручную


Хорошо, теперь пришло время для запуска
и для того чтобы мы могли посмотреть
что мы получаем в итоге.
Запустите плагин с помощью atlas-run,
а затем мы используем curl для поиска пользователей.
Дождитесь запуска atlas-run
и пока полностью запустится джира,
а затем отправьте поисковый запрос к нашему рест-сервису поиска пользователей.



curl --user admin:admin http://localhost:2990/jira/rest/kitchenduty/1.0/user/search?query=adm
[{"text":"admin","id":"admin"}]


поскольку мы указали query=adm,
мы хотим получить список всех пользователей,
имя которых начинается с adm.
В ответе мы видим, что на текущий момент в джире существует только один такой пользователь
и это admin.



You have completed Step 2. Good Job! You can check out this step's solution on GitHub
https://github.com/comsysto/kitchen-duty-plugin-for-atlassian-jira/tree/master/step-02-kitchen-duty-plugin


На рисунке зеленым цветом показаны компоненты, которые мы реализовали в этом разделе руководства.





=====================================================================================

# User Search JS Controller



в предыдущем разделе мы реализовали конечную точку рест
поиска пользователя

которую мы теперь будем использовать
для нашего контроллера жс
возможно вы захотите еще раз проверить
Story Workshop чтобы вспомнить что за интерфейс мы хотим сделать здесь.

Первое что на интересует это версия AUI и JIRA

Итак мы хотим использовать виджет Select2 из AUI Framework
Во первых вы должны знать что каждая версия jira
имеет aui
предварительно упакованный в определенную версию.
Поэтому мы будем использовать ту версию которая соответсвует нашей джире
и не будем добавлять вторую версию этого.

Не стоит такое делать, правда.

Поэтому нам необходимо выполнить команду atlas-run
и подождать пока жира запустится.
Теперь откройте консоль браузера и введи такую команду:


AJS.version
5.7.31

в выводе команды будет напечатана точна версия ajs
используемая в целевой версии джиры для которой мы пишем плагин.
На этом этапе вам действительно нужно подумать о диапазоне версий
которые вы хотите поддерживать для своего плагина.
Если вы хотите поддерживать более старые версии джира,
то вам нужно знать самую низку версию AUI и писать код с учетом 
обратной совместимости.
Чтобы упростить ситуацию, мы предполагаем,
что наш плагин будет совместим, начиная с Jira 7.0
со всеми версия джира до последней (на текущий момент это 7.1)

Хорошо, теперь откройте документацию AUI 
для верси 5.7.31
и выберите AUI Seleect 2 
и прочтите документацию для этого компонента.
Ладно, давайте не будем обижать людей которые пишут документацию,
но она немножко того,
и мне потребовалось некоторое время,
чтобы найти простой способ использовать Select 2 так,
как я хочу, чтобы он себя вел.
Вы же только что прочитали документацию
сами и потому можете если захотите написать такой код который
вам понравится.

# А как насчет JS фреймворков, шаблонизаторо в и jquery?

Хорошо, давайте поговорим о нашем виджете.
Как мы напишем наш код на жс?
Есть ли какой-нибудь шаблонизатор?
Любые watchers  and/or фреймворки?

Позвольте мне рассказать вам котороткую историю о Jira и джаваскрипт.

Итак, мы уже использовали такую штуку как AJS.version ранее
так что вы возможно уже задались вопросом,
что же это на самом деле.
JIRA предоставляет легкий jquery,
который привязан к AJS,
в нем нет ни $ ни jQuery
и даже если он был у вас, все равно рекомендуется использовать
префикс AJS

Итак вот основной пример:

var theFooElement = AJS.$('#foo');
undefined

Этот фрагмент кода выберет элемент DOM с идентификатором foo
(когда такого элемента нет, он вернет undefined)
Хорошо, я понимаю что это вам кажется необычным и незнакомым,
но мы привыкнем к этому.

Так насчет шаблонизатором?
Существует сой,
который по сути представляет собой Google Closure Template Framework,
и этот шаблонизатор можно использовать как на стороне сервера 
так и на строне клиента.
Мы будем придерживаться velocity для шаблонов находящихся на стороне сервера,
но вы если захотите можете полность перейти на сой.
На стороне клиента мы будем использовать сой-шаблоны.


Минимализм это прекрасно.
Избегайте использования больших жс фреймворков,
которые регистрируют глобальные переменные,
вносят зависимости
и имеют тенденцию перезаписывать данные,
и таким образом могут нарушить функциональность джира.
В большинстве случаев AUI уже предоставляет собой достаточно средств для решения ваших задач.


Хорошо, давайте подведем небольшие итоги по поводу того, что мы собираемся использовать:
1 - легкий jQuery, доступ к которому мы имеем через префикс AJS
2 некоторые методы фреймворка AUI, такие как AJS.getI18nText
3 - сой шаблоны (Google Closure Template Framework)
4 HTML страницу с шаблонизатором velocity которая будет генерироваться на стороне сервера,
и которую мы особенно много использовать

## Базовая настройка зависимостей и контроллера JS

Теперь мы создаем наш файл JS-контроллера
в /src/main/resources/js/kitchen-duty-plugin-planning-page-controller.js

внутри мы поместим некоторый простой код-заглушку,
который вызовет покажет сообщени при загрузке страницы,
просто чтобы показать что все работает.

src/main/resources/js/kitchen-duty-plugin--planning-page-controller.js

AJS.toInit(function(){
   require(['aui/flag'], function(flag) {
       var myFlag = flag({
           type: 'success',
           title: 'Kitchen Duty Plugin',
           body: 'JS Controller is working'
       });
   });
});


Наш новый js контроллер не будет работать из коробки.
нам нужно определить ресурс в atlasian-plugin.xml для нашего контроллера.

src/main/resources/atlassian-plugin.xml

<web-resource key="kitchen-duty-plugin-resources--planning-page" name="kitchen-duty-plugin Web Resources for Planning Page">
	<dependency>com.atlassian.auiplugin:ajs</dependency>
	<resource type="download" name="kitchen-duty-plugin--planning-page-controller.js" location="/js/kitchen-duty-plugin--planning-page-controller.js"/>
	<context>kitchen-duty-plugin</context>
</web-resource>



Мы уже определили ресурс, но джира не знает,
когда использовать (загружать) ресурсы, которые мы определили.
Мы будем использовать PageBuilderService,
чтобы сообщить JIRA,
когда загрузать наши ресурсы

возможно вы захотите попробовать #require в шаблонах Velocity,
но PageBuilderService всегда работал лучше для меня.

Мы добавляем некоторые дополнительные зависимости,
чтобы иметь возможность использовать PageBuilderService.
Добавьте эти зависимости в ваш pom.xml


pom.xml

<dependency>
	<groupId>com.atlassian.templaterenderer</groupId>
	<artifactId>atlassian-template-renderer-api</artifactId>
	<version>1.5.7</version>
	<scope>provided</scope>
</dependency>
<dependency>
	<groupId>com.atlassian.plugins</groupId>
	<artifactId>atlassian-plugins-webresource</artifactId>
	<version>3.3.3</version>
	<scope>provided</scope>
</dependency>


Мы уже узнали, что нам нужно добавить аннотацию @ComponentImport 
где нибудь в нашем приложении,
чтобы получить зависимости из других бандлов osgi.
Вот почему мы просто делаем это в нашем файле MyPluginComponentImpl.java
который мы будем (неправильно)
использовать для этой цели отныне.

src/main/java/com/comsysto/kitchen/duty/impl/MyPluginComponentImpl.java


import com.atlassian.webresource.api.assembler.PageBuilderService;

...

@ComponentImport
private PageBuilderService pageBuilderService;




Теперь, когда PageBuilderService можно использовать,
нам нужно сообщить нашему webwork actions о загрузке наших ресрусов.



src/main/java/com/comsysto/kitchen/duty/webwork/KitchenDutyPlanningWebworkAction.java


package com.comsysto.kitchen.duty.webwork;

import com.atlassian.webresource.api.assembler.PageBuilderService;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import com.atlassian.jira.web.action.JiraWebActionSupport;

import javax.inject.Inject;
import javax.inject.Named;

@Named 
public class KitchenDutyPlanningWebworkAction extends JiraWebActionSupport
{
    private static final Logger log = LoggerFactory.getLogger(KitchenDutyPlanningWebworkAction.class);

    @Inject 
    private PageBuilderService pageBuilderService;

    @Override
    public String execute() throws Exception {
        pageBuilderService.assembler().resources().requireWebResource(
           "com.comsysto.kitchen-duty-plugin:kitchen-duty-plugin-resources" 
        ).requireWebResource(
           "com.comsysto.kitchen-duty-plugin:kitchen-duty-plugin-resources--planning-page"
        );

        return "kitchen-duty-planning-success";
    }

    public void setPageBuilderService(PageBuilderService pageBuilderService) { 
        this.pageBuilderService = pageBuilderService;
    }
}

Определим нашу KitchenDutyPlannginWebworkAction как @Named компонент
Будем использовать @Inject чтобы подключить PageBuilderService.
Мы сообщаем PageBuilderService() о необходимости
загрузить наш общий бандл с ресурсами
и наш planning-page бандл, который содержит контроллер.
Шаблон который мы используем для require это "pluginKey:resourceKey"

И конечно нам нужен сеттер для setter injection PageBuildService.


После запуска жиры, при помощи команды atlas-run
(вы должны перезапустить ранее уже запущенный atlas-run,
поскольку мы кое-что добавили в pom.xml)
вы должны увидеть зеленое сообщение об успехе,
которое появится в правом верхнем углу при просмотре страницы планирования


http://localhost:2990/jira/secure/KitchenDutyPlanningWebworkAction.jspa

https://comsysto.github.io/kitchen-duty-plugin-for-atlassian-jira/images/doc/step-03-js-controller-working.gif



## Следующий этап - это настройка виджета Select 2 для поиска пользователей
Прежде чем мы начнем, вы можете (но не обязательно)
подробно прочитать о AUI макетах, формах (layout, forms) и про soy

Что такое сой? 
Это шаблонизатор,
который мы можем использовать как на стороне сервера
так и на стороне клиента.
Здорово!
Но нам понадобится преобразователь,
чтобы преобразовывать шаблоны сой в джаваскрипт.
Чтобы иметь возможностль использовать их в нашем жс-контроллере.

Вот краткое резюме того, что мы будем делать сейчас:
создадим сой шаблон для поиска пользователя,
определим transformer, чтобы иметь возможность использовать сой шаблон в жс контроллере
напишем некоторый код жс для использовать виджета aui select
с нашим ресурсом REST поиска пользователей (смотрите предыдущий раздел)

Создайте файл сой шаблона в src/resources/templates-soy/kitchen-duty-planning.soy
и вставьте туда следующий код

Этот файл позже будет содержать более одного шаблона для каждо варианта использования.
Но у нас будет один такой файл на страницу.

src/main/resources/templates-soy/kitchen-duty-planning.soy

{namespace JIRA.Templates.KDP} 
/**
 * Kitchen Duty Planning Page - User Search Template
 */
{template .userSearch} 
<form class="aui" id="kdp-user-select-form"> 

    <div class="field-group">
        <label for="kdp-user-select">Users</label>
        <div id="kdp-user-select"  
                 name="kdp-user-select"
                 class="kdp-aui-select"
                 multiple=""
                 placeholder="start typing a username ...">
        </div>
        <div class="description">The users that should have kitchen duty.</div>
    </div>
    <div class="buttons-container">
        <div class="buttons">
            <input class="button submit" type="submit" 
                   id="kdp-user-select-save-button"
                   value="show"/>
        </div>
    </div>

</form>
{/template}

Мы определяем пространство имен с именем JIRA.Templates.KDP для наших шаблонов.
Позже мы сможем получить доступ ко всем нашим шаблонам
при помощи namespace.templateName

Наш первый шаблон будет называться .userSearch
и будет содержать html код
который мы позже будем использовать из нашего контроллера жс

Поля ввода и виджеты AUI будут правильно отображаться только если мы
поместим их в элемент form с классом
который называется aui
Мы определим идентификатор для формы чтобы
позже иметь возможность обратиться к ней через jquery.

Это контейнер для виджета AUI Select 2.
Мы используем div с идентификатором kdp-user-select.
Не забудьте multiple
чтобы указать виджету
в init
что это поле будет принимать несколько значений.

Последнее что нам нужно, это кнопка отправки формы.
Мы также даем кнопке уникальный идентификатор, чтобы 
иметь возможно обратиться к ней затем из jQuery.


Измените шаблоны velocity чтобы нас был один контейнер div
с идентификатором,
который мы сможем использовать для динамической вставки контента,
который будет рендерится js контроллером.


src/main/resources/templates/kitchen-duty-planning-webwork-module/kitchen-duty-planning-success.vm

<html>
<head>
    <title>$i18n.getText("kitchen-duty-plugin.admin.planning.page.title")</title>
    <meta name="decorator" content="atl.admin">
</head>
<body>
<h1>$i18n.getText("kitchen-duty-plugin.admin.planning.page.headline")</h1>

<div id="kdp-planning-page-container"></div> 

</body>
</html>


Единственно, что изменяем в нашем шаблоне Velocity,
это помещаем в него контейнер div,
который мы будем использоваться для добавления отрендереных шаблонов.



Измените js контроллер для визуализации soy шаблона
инициализируйте виджет AUI Select 2 для работы с нашим рест-ресурсом поиска пользователей.


src/main/resources/js/kitchen-duty-plugin--planning-page-controller.js


var showSuccessFlag = function(message) { 
    require(['aui/flag'], function(flag) {
        var myFlag = flag({
            type: 'success',
            title: 'Kitchen Duty Plugin',
            close: 'auto',
            body: message
        });
    });
};

var initUserSearch = function(restUrl) { 
    var templateUserSearch = JIRA.Templates.KDP.userSearch(); 

    var auiUserSelectOptions = {
        ajax: {
            url: function () {
                return restUrl + '/user/search'; 
            },
            dataType: 'json',
            delay: 250,
            data: function (searchTerm) {
                return {
                    query: searchTerm
                };
            },
            results: function (data) {
                return {
                    results: data
                };
            },
            cache: true
        },
        minimumInputLength: 1,
        tags: 'true' 
    };

    /* INIT TEMPLATES AND WIDGETS */

    AJS.$('#kdp-planning-page-container').append(templateUserSearch); 
    AJS.$('#kdp-user-select').auiSelect2(auiUserSelectOptions); 
    AJS.$('#kdp-user-select-form').submit(function (e) { 
        e.preventDefault();
        AJS.$(AJS.$('#kdp-user-select').select2('data')).each(function () { 
            showSuccessFlag(this.id);
        });
    });
};

AJS.toInit(function(){ 
    AJS.log('KDP: Planning Page Controller initializing ...');
    var baseUrl = AJS.params.baseURL; 
    var restUrl = baseUrl + '/rest/kitchenduty/1.0';

    initUserSearch(restUrl); 
});




Мы помещаем message flag в функцию,
чтобы мы могли легко создавать success flags везде где захотим.
Мы определяем функцию инициализации для нашего кода поиска пользователя.
Там мы инициализируем все что нам нужно для AUI Select 2.
Первое что мы делаем,
это получаем сой шаблон
userSearch и можем его использовать.
auiUserSelectOptions это опции AUI Select 2,
о которых вы можете прочесть в документации.
Здесь мы определяем URL,
где поле AUI Select 2
должно запрашивать имена пользователей.
Остальные варианты необходимы и должны быть понятны.
Эта опция сообщает AUI Select 2,
что мы хотим иметь теги,
которы должны быть removable.
Теперь мы добавляем наш обработанный сой шаблон
userSearch в DOM
в наш специальный div-контейнер,
который мы определили ранее.
сразу после этого мы инициализируем поле AUI Select 2
для userSearch с ранее определенными параметрами.
Теперь мы подключаем submit button 
кнопк отправки формы
и запрещаем default-form-submit
В противном случае форма вызовет запрос POST/GET который нам не нужен.

Позже мы will hook код, чтобы сохранить selection здесь,
но это произойдет в следующий разделах руководства.
На данный момент
у нас порядок с тем что каждый выбранный юзернейм отображается с success флагом.
Select 2('data')
возвращает массив со всеми выбранными элементами

Функция AJS.toInit это маленький помощник,
который выполняет обратный вызов при загрузке AUI Framework.
Мы помещаем туда все вызова наших функций инициализации.


Мы получаем baseUrl из AUI Framework.
Если вы вызовете AJS.params.baseURL ранее,
за пределами хелпера AJS.init,
то вы получите неопределенные ошибки.
В зависимости от baseUrl
мы созданем наш базовый restUrl,
который является общим для всех наших точек доступа нашего рест-апи.

Наконец мы вызываем функцию initUserSearhc и передаем ей restUrl.

Теперь нам нужно сказать джире, 
что мы хотим использовать сой в нашем клиентском коде
и определить transformer
и соевый шаблон для загрузки.
Также нам нужно определить зависимости AUI для загрузки.


src/main/resources/atlassian-plugin.xml


<?xml version="1.0" encoding="UTF-8"?>

<atlassian-plugin key="${atlassian.plugin.key}" name="${project.name}" plugins-version="2">
...
  <web-resource key="kitchen-duty-plugin-resources--planning-page" name="kitchen-duty-plugin Web Resources for Planning Page">
    <dependency>com.atlassian.auiplugin:ajs</dependency>
    <dependency>com.atlassian.auiplugin:aui-select2</dependency> 
    <dependency>com.atlassian.auiplugin:aui-experimental-soy-templates</dependency> 
    <transformation extension="soy"> 
      <transformer key="soyTransformer">
        <functions>com.atlassian.confluence.plugins.soy:soy-core-functions</functions>
      </transformer>
    </transformation>
    <resource type="download" name="kitchen-duty-planning-soy.js" 
              location="templates-soy/kitchen-duty-planning.soy"/>
    <resource type="download" name="kitchen-duty-plugin--planning-page-controller.js"
              location="/js/kitchen-duty-plugin--planning-page-controller.js"/>
    <context>kitchen-duty-plugin</context>
  </web-resource>
...
</atlassian-plugin>


Мы указываем зависимости для виджета AUI Select 2,
чтобы JIRA предоставила его при загрузке нашего пакета.

Мы также указываем особую зависимость для soy,
как это указано в документации (хотя я думаю что это ничего не делает)

Теперть мы указываем transformer,
который преобразует соевые шаблоны в джаваскрипт.
Он преобразует все файл с расширений soy в javascript.
(но все-таки не все. смотрите следующий пункт)

Мы указываем другой ресурс загрузки.
Это файл джаваскрипт,
который содержит все наши соевые шаблоны для страницы планирования.
location - это путь к исходному файлу,
который является соевым.
соевый файл будет динамически преобразован
в джаваскрипт 
с помощи ранее определенного трансоформера (преобразователя)



Теперь перезапустите atlas-run и обновтие shift-reload страницу
и вы должны увидеть это:

https://comsysto.github.io/kitchen-duty-plugin-for-atlassian-jira/images/doc/step-03-user-search-select2.gif


Хорошо теперь мы можем искать пользователей и добавлять или удалять их наше поле ввода.
В следующей главы мы займемся Kitchen Duty Planning REST Resources,
которые позволят нам сохранять наш выбор.

Дальнейшие главы будут посвящены
контроллеру js,
сохраняющему выборы и загружающему AUI Select 2 c нашим сохраненным выбором.



You have completed Step 3. Good Job! You can check out this step's solution on GitHub
https://github.com/comsysto/kitchen-duty-plugin-for-atlassian-jira/tree/master/step-03-kitchen-duty-plugin


На рисунке зеленым цветом показаны компонеты, которые мы реализовали в этом разделе.

# Kitchen Duty Planning REST Resource

По сути мы создадим рест ресурс
для планирования работы кухни
и рест модель
для планирования работы в кухне,
которая сохранит выбранные недели и пользователей в базе данных

Вы также можете захотеть взглянуть в Story Workshop
чтобы повторить что мы собираемся сделать сейчас.

Эта глава сильно зависит от конфигурации,
сделанной в шаге 2

## что именно нам нужно?
После некоторого серьезного использования мозга о том,
как хранить фактические данные о том кто должен дежурить на курне на какой-то неделе?
Я придумал модель данных показанную справа.

Неделя будет будет фактическим номером недели в году,
и нам плевать какой это год.
так что если вам был кухонный долн на 24-ой неделе в этом гоуд
он будет и через несколько лет.
Немного подумав о том, как я хочу использвоать это в контроллере жс,
я в оновном выберу неделю с помощь средства выбора даны
и вычислю номер недели во внешнем интерфейсе
при помощи moment.js

Пользовательская часть очевидна.
Нам нужно сохранить список имен пользователей в базе данных,
относительно недель, когда эти пользователя будут дежурить на кухне.


@ManyToMany

Что касается конечно точки рест,
я хочу иметь урл адерс,
потокроу я могу отправлять запросы типа GEt
на что-то вроде
planning/week/15/users
и получать список пользователей.

Очевидно, что обратрым вариантом будет запрос PUT
для пользователей
/planning/week/15/users

со списком пользователей в качестве тела запроса
дял сохранения списка пользователей для недели с номером 15


И для нашей расширенной цели нужен запрос GET 
на /planning/user/bob/weeks
который должен возвращать список номеро недель у который у боба есть дежурство на кухне

## Как именно мы сохраняем наши данные?

Хорошо, мы знаем, как мы хотим использовать нашу конкретную точку рест с точки зрения контроллера жс.
И у нас есть какая-то подсказка о том, как данные должны храниться теоретически.
Существует две основые возможности для хранения данные в жира.
Используя PluginSettingFactory у вас есть способ сохранить данные
Но у вас нет запросов
и вы не можете строить связи между данными,
как вы этом можете делать с реальной базой данных.
Так что это хорошо, если вы хотите хранить какие-то очень простые данные,
когда вы всегда знаете как извлечь данные из PluginSettingsFactory
(например при помощи фиксированного имени ключа)

Второй способ хранить данные это Active Objects
которые позволяют вам использовать реальную базу данных
с интерфейсом ORM
И это безусловно выбор, который мы хотим сделать здесь.
Поэтому вы можете немного прогуляться по документции по Active Objects,
чтобы понять ее, или просто продолжить чтение,
так как я уже познакомлю вас с этой штукой.

## Enabling Active Objects on our Project

Все что я напишу об активных объектах
взято из руководства по разработке плагина с активными объектами 
и руководства по началу работы с активными объектами.

Прежде всего добавьте зависимость Active Objects
в ваш pom.xml

<dependency>
   <groupId>com.atlassian.activeobjects</groupId>
   <artifactId>activeobjects-plugin</artifactId>
   <version>1.1.5</version>
   <scope>provided</scope>
</dependency>


Нам нужен ComponentImport для ActiveObjects,
который будет объектом, через который мы будем получать доступ к базе данных


src/main/java/com/comsysto/kitchen/duty/impl/MyPluginComponentImpl.java

package com.comsysto.kitchen.duty.rest;

import com.atlassian.activeobjects.external.ActiveObjects;
import com.atlassian.plugin.spring.scanner.annotation.imports.ComponentImport;
...

@Named ("myPluginComponent")
public class MyPluginComponentImpl implements MyPluginComponent {
   ...
   @ComponentImport
   private ActiveObjects activeObjects;
   ...
}


Теперь мы создаем нашу первую Entity Week
и определять неделю как целое число через getter/setter



src/main/java/com/comsysto/kitchen/duty/ao/Week.java

package com.comsysto.kitchen.duty.ao;

import net.java.ao.Entity;
import net.java.ao.Preload;

@Preload 
public interface Week extends Entity { 
    Integer getWeek();

    void setWeek(Integer week);
}

Мы аннотируем наш класс при помощи @Preload
что я думаю делает его загрузку не ленивой,
но вы должны прочитать документацию если хотите узнать подробности.
Важно то, что мы расширяем Entity,
что делает наш класс автоматически имеющим метод getId()
поэмуто нам просто нужно определить наши поля,
которые в настоящее время состоят только из 
номера недели (номер недели в году)

Мы только что создали Week Entity сейчас
и нам нужно сообщить модулю активных объектов нашего плагина,
где он может найти нашу сущность.
Поэтмоу мы добавляем раздел в файл atlassian-plugin.xml
со списком сущностей.
Возможно есть какая-то аннотация, чтобы пропустить этот шаг,
но поскольку ничерта нет подробной документации по этому вопросу,
то я не знаю. 
Если вы сами узнаете, то скажите мне пожалуйста.

src/main/resources/atlassian-plugin.xml

<ao key="ao-module">
    <description>The module configuring the Active Objects service used by this plugin</description>
    <entity>com.comsysto.kitchen.duty.ao.Week</entity>
</ao>

## Creating a basic Week Entity and a persist-Endpoint

Мы проворные программисты agile coders
и хотим видеть основные результаты того, что мы сделали.
Поэтому мы создадим очень простую конечную точку доступа к апи,
назовем ее persitsTest
и будем при помощи этого эндпоинта тестировать ORM,
которая в данном случае представлена Active Objects.

Мы избавимся от этого эндпоинта позже,мы просто используем ее,
чтобы понять, как должны работать активные объекты.

src/main/java/com/comsysto/kitchen/duty/rest/KitchenDutyPlanningResource.java


package com.comsysto.kitchen.duty.rest;

import com.atlassian.activeobjects.external.ActiveObjects;
import com.atlassian.jira.bc.user.search.UserSearchService;
import com.atlassian.plugins.rest.common.security.AnonymousAllowed;
import com.atlassian.sal.api.transaction.TransactionCallback;
import com.comsysto.kitchen.duty.ao.Week;

import javax.inject.Inject;
import javax.inject.Named;
import javax.servlet.http.HttpServletRequest;
import javax.ws.rs.GET;
import javax.ws.rs.Path;
import javax.ws.rs.Produces;
import javax.ws.rs.QueryParam;
import javax.ws.rs.core.Context;
import javax.ws.rs.core.MediaType;
import javax.ws.rs.core.Response;
import java.util.ArrayList;
import java.util.List;

@Named
@Path("/planning")
public class KitchenDutyPlanningResource {

    private ActiveObjects activeObjects; 

    @Inject 
    public KitchenDutyPlanningResource(ActiveObjects activeObjects) {
        this.activeObjects = activeObjects;
    }

    public KitchenDutyPlanningResource() {
    }


    @GET
    @Path("/persistTest") 
    @Produces({MediaType.APPLICATION_JSON})
    @AnonymousAllowed
    public Response persistTest() {
        activeObjects.executeInTransaction(new TransactionCallback<Week>() 
        {
            @Override
            public Week doInTransaction() 
            {
                final Week testWeek = activeObjects.create(Week.class); 
                testWeek.setWeek(42);
                testWeek.save();
                return testWeek;
            }
        });
        return Response.ok("ok").build();
    }


    @GET
    @Path("/health")
    @Produces({MediaType.APPLICATION_JSON})
    @AnonymousAllowed
    public Response health() {
        return Response.ok("ok").build();
    }

}


Определите ActiveObjects как свойство,
чтоы мы могли его использовать.
Добавьте @Inject в конструктор,
чтобы мы внедрили его через DI.
Определите эндпоинт при помощи path persistTest
и метода @Get, который мы будем использовать для игры с ORM.
Мы начинаем транзакцию
... this should be done with Annotations ... but we have to deal with it ...
и здесь нам нужно определить наш тип возврата,
который в нашем случае Week.
Вы также можете установить здесь значение Void.
Нам нужно переопределить doInTransaction
и внутри мы можем выполнять реальные действия
с объектами ORM Active Objects.
Очевидно, что мы здесь могли бы определить больше методов
для обработки отката транзакций,
и обычно мы должны обернуть все эти махинации
в блок try-catch
и сделать правильную обработку исключений.
Но поскольку в настоящее время мы просто играем,
то все в порядке.

Теперь, когда мы глубоко внутри кроличьей норы,
мы можем нчать сохранять наш первый объект Week,
Мы вызываем ActiveObejcts.create() и передаем Class-Type
после этого мы устанавливаем значение нашей недели и вызываем save().
Теперь наша неделя сохраняется в базе данных, банзай!



Хорошо, теперь запустите JIRA с помощью atlas-run,
но запишите выходной файл с имень jira-stdout.log,
чтобы мы могли грепать его позже.
У вас должен быть установлен tee.

atlas-run | tee jira-stdout.log
...
[INFO] [talledLocalContainer] INFORMATION: Server startup in 38377 ms
[INFO] [talledLocalContainer] Tomcat 8.x started on port [2990]
[INFO] jira started successfully in 58s at http://localhost:2990/jira
[INFO] Type Ctrl-D to shutdown gracefully
[INFO] Type Ctrl-C to exit


Откройте другой терминал,
чтобы выполнить некоторые команды CURL,
и как только JIRA запустится,
выполните GET запрос к нашей конечной точке persistTest.





curl --user admin:admin -H 'Content-Type: application/json' http://localhost:2990/jira/rest/kitchenduty/1.0/planning/persistTest
ok


Теперь наш объект Week с неделей номер 42 должен быть сохранен в таблице Week базы данных.

