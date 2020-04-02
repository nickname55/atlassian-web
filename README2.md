Developing a JIRA add-on like it’s 2016: Part Two

Добро пожаловать!
В первой части этой серии статей
мы рассмотрели
все новые инструменты
и детали сборки проекта,
необходимые для улучшенного способа создания джира плагинов.
Если вы помните, наша цель состоит в том,
чтобы написать дополнение,
которое мы можем легко преобразовать
из серверного в облачное.
Это требует от нас соблюдения двух руководящих приципов:

Доступ к данным с сервера возможен только через 
очень точно определенный REST API.
JSON, возвращаемый этим API,
должен быть стандартным,
чтобы не имело значения,
исходит ли он из ресурса REST,
определенного на сервере JIRA
в файле atlassian-plugin.xml
или из ресурса REST в автономном сервере atlassian-connect.

Пользовательский интерфейс будет рендериться исключительно
на стороне клиента
с помощью автономных компонентов,
не зависящих от JavaScript идущего вместе с JIRA.

Сегодня мы рассмотрим,
как некоторые из этих компонентов
реализованы
на стороне клиента
с использованием React
и ES6.

# React to the rescue!
Построение сложных пользовательских интерфейсов
исключительно на стороне клиента
значительно улучшилось за послдение годы.
Прошли времена, когда код джаваскрипт
напоминал собой спагетти.
Теперь легко вы можете легко писать
простые и чистые 
повторно используемые компоненты,
которые легко тестировать.


Мы рассмотрим очень сложную часть дополнения NPS
для JIRA, а именно страницу с отчетом.

https://blog.developer.atlassian.com/wp-content/uploads/dac-import/nps-reports.png

На этой странице много чего происходит)

- у нас есть заголовок страницы, на котором расположены
различные элементы управления.
Эти элементы управления представляют собой фильтры для отбора задач
по различным параметрам.

- у нас есть отчет о баллах, показывающий общий балл NPS
и проценты от общего количества

- у нас есть раздел с вкладками, содержащий подробную информацию
и отчет sentiment word cloud report

- и отчет о потраченном времени, расположенный в нижней части
страницы (и отрисовывающийся при помощи диаграммы JS)



Давайте посмотрим на компонент ReportsPage React.
Кажется что это довольно сложно,
но на самом деле это не так.

'''

//imports excluded for brevity
const ReportsPage = (props) => (
  <div>
    <ReportsHeader />
    <section style={style.content}>
      <Aui.Group>
        <Aui.Item>
          <h2>{i18n.getText('survey.plugin.reports.score')}</h2>
          <ScoresReport context={props.context} />
        </Aui.Item>
        <Aui.Item>
i.Tabs>
            <Aui.Tab id="details-report" title={i18n.getText('survey.plugin.reports.details')}>
              <DetailsPanel context={props.context} />
            </Aui.Tab>
            <Aui.Tab id="sentiment-report" title={i18n.getText('survey.plugin.reports.sentiment')} lazyRendered >
              <SentimentReport context={props.context} />
            </Aui.Tab>
          </Aui.Tabs>
        </Aui.Item>
      </Aui.Group>
      <h2>{i18n.getText('survey.plugin.reports.overtime')}</h2>
      <OvertimeReport context={props.context} />
    </section>
  </div>
);

ReportsPage.propTypes = {
  context: PropTypes.object.isRequired,
};

//ignore this for now. It's Redux which we'll cover later
const mapStateToProps = (state) => ({
  context: state.context,
});

export default connect(mapStateToProps)(ReportsPage);

'''

Даже для тех, кто не знаком с кодом,
довольно легко понять,
что здесь происходит и как страница отчетов
разбита на несколько подкомпонентов React.

- ReportsHeader
- ScoresReport
- DetailsPanel
- SentimentReport (the word cloud not shown in the screenshot above on the second tab)
- OvertimeReport

Некоторым синтаксис может показаться немного незнакомым,
но благодаря возможностям Babel
мо можем использовать все возможности ES6 и React JSX:

HTML часть это на самом деле React JSX,
который компилируется в JavaScript.

Как компонент ReportsPage определяется как функция
без сохранения состояния
с помощью ES6 arrow notation.
И, наконец, мы используем ES6 exports
для экспорта этого компонента,
чтобы сделать его доступным для импорта в других компонентах.

# Working with 3rd party libraries
# Работа со сторонними библиотеками

Ранее мы упоминали,
что в наших отчетах для построения графиков
используется диаграмма созданная при помощи Chart JS.
Это может быть немного сложно с реактом,
так как реакт очень жестко контролирует DOM
и то как им манипулируют.
Вот пример того, как компонент NPS Doughnut (пончик),
который является субкомпонентом Scores report,
работает чтобы нарисовать пончиковую диаграмму:


'''

//imports excluded for brevity
class NPSDoughnutChart extends Component {
  componentDidMount() {
    this.config = {
      type: 'doughnut',
      data: { //chart js doughnut config options excluded for brevity };

    this.doughnutChart = new Chart(this.canvas, this.config);
  }

  componentDidUpdate() {
    this.config.data.datasets[0].data = this.getData();
    this.doughnutChart.update();
  }

  getData() {
    const npsInfo = this.props.npsInfo;
    if (npsInfo.total !== 0) {
      return [npsInfo.promoters.count, npsInfo.passives.count, npsInfo.detractors.count];
    }
    return [1, 1, 1];
  }

  render() {
    const score = Math.round(this.props.npsInfo.score * 100);

    return (
      <div className="nps-score-doughnut" style={Object.assign({}, style.container, this.props.style)}>
        <div ref={(c) => { this.innerContainer = c; }} style={style.innerContainer}>
          <canvas ref={(el) => { this.canvas = el; }} width="100%" height="100%" />
          <h2 style={style.score}>{score}</h2>
        </div>
      </div>
    );
  }
}
NPSDoughnutChart.propTypes = {
  npsInfo: PropTypes.shape({
    score: PropTypes.number.isRequired,
    responses: PropTypes.number.isRequired,
    promoters: PropTypes.object.isRequired,
    passives: PropTypes.object.isRequired,
    detractors: PropTypes.object.isRequired,
  }),
  style: PropTypes.object,
};

export default NPSDoughnutChart;


'''

Давайте сначала посмотрим на функцию render():

Она визиализирует элемент canvas
и помещает React ref в this.canvas.

Это также рендерит h2 , который содержит total scope (общий счет)

Мы также используем встроенные стили для наших компонентов,
что рекомендовано React.
style.scope попадает к нам из импортированного модуля,
который просто определяет атрибуты для стиля h2

После выполнения render() 
React вызовет componentDidMount(),
в тот момент когда отображаемый контент
вставляется в реальный DOM
из shadow DOM.
В этом методе мы теперь можем инициализировать все параметры
диаграммы Chart JS, используя
ссылку ref на canvas DOM element.


Единственное, что нам нужно сделать,
это убедиться, что мы обновляем диаграмму через componentDidUpdate() 
всякий раз, когда этот компонент получает новые реквизиты,
которые могут изменить диаграмму (например, 
если пользователь изменил фильтр в ReportsHeader)

Вот и все!

# Тестирование

Традиционно, плагины JIRA тестировались
при помощью Web-driver тестов
и qunit тестов в браузере.
Оба этих подхода имеют тенденцию быть довольно медленными,
поскольку они требуют, чтобы JIRA была запущена
и чтобы браузер был настроен.

Это не является большой проблемой при разработке,
но может быть проблематичным при настройке
и может быть причиной медленного выполнения в CI.

Для плагина NPS интерфейсный код тестируется
с использованием облегоченной фреймворка для
тестирования. Фреймворк называется Mocha JS.
Мы также используем Sinon JS для моков.
Вот пример теста, чтобы вы могли убедиться,
что overtime chart может обрабатывать различные
ответы от сервера.


'''
//imports excluded for brevity

const sampleResponse = {
  start: 1453123055079,
  end: 1453727855079,
  prettyDate: '25/Jan/16',
  jql: '/issues/?jql=project+%3D+ASDF+AND+cf%5B10000%5D+is+not+EMPTY+' +
  'AND+created+%3E%3D+%222016-01-18+20%3A17%22+AND+created+%3C%3D+%222016-01-25+20%3A17%22',
  npsScore: -0.45833337,
  responseCount: 24,
};

function renderReport(resolve) {
  const context = {
    baseUrl: '',
    jql: 'project='TEST'',
    projectKey: 'TEST',
    surveyId: 'sample-survey-id',
  };

  return TestUtils.renderIntoDocument(<OvertimeReport context={context} ajaxDataRenderResolve={resolve} />);
}

describe('Over time Report', () => {
  afterEach(() => {
    fetchMock.restore();
  });

  //other tests testing error responses excluded for brevity
  it('Data from server renders chart', () => {
    let el;
    fetchMock.mock('^/rest/nps/1.0/reports/TEST/sample-survey-id/overtime', {
      status: 200,
      body: { data: [sampleResponse] },
    });
    return new Promise((resolve) => {
      el = renderReport(resolve);
    }).then(() => {
      const canvas = TestUtils.scryRenderedDOMComponentsWithTag(el, 'canvas');
      expect(canvas.length).to.be(1);
    });
  });
});

'''
В связи с асинхронной природой вызова REST,
нам нужно использовать Promises,
которые разрешаются самим компонентом
через ajaxDataRenderResolve prop,
прежде чем мы запустим assertions.
Это немного bit of a smell но позже мы увидим,
как Redux может помочь удалить нам этот bit of a smell.

Несколько других интересных вещей, которые мы можем отметить:
Мы используем fetch Mock для имитации REST-вызовов на сервер.
Утилиты React TestUtils позволяют нам отображать
и запрашивать компоненты React во время теста.

# Build integration

Выполнение этих тестов невероятно быстро,
поскольку мы выполняем их не в браузере,
а в jsdom - node js имплементации DOM.
Это требует небольшой настройки,
которые выполняется в модуле test-setup module.
Изучив наш скрипт test в package.json
и первой статьи данной серии статей,
мы видим что нам нужен этот модуль: test-setup

'''
"test": "./node_modules/.bin/mocha --compilers js:babel-core/register "./src/**/*test.js" --colors --require test-setup",

'''


Этот модуль просто содержит настройки для jsdom,
и мы также настраиваем наши polyfills:


'''

var jsdom = require('jsdom').jsdom;

global.document = jsdom('<!doctype html><html><body></body></html>');
global.window = global.document.defaultView;
global.navigator = global.window.navigator;

require('babel-polyfill');
require('isomorphic-fetch');

'''

Теперь вы можете запускать тесты из командной строки
с помощью команды 'npm run test'
Для IntelliJ также есть Mocha.
В первой части этой серии статей вы также увидите сценарий
test-maven, который мы вызываем из Maven
и который использует Mocha test reporters
для создания XML-файла с результатами теста junit,
которы может быть затем проанализирован в Bamboo.

# I18n Интернационализация

Существует несколько платформ React i18n.
Однако большиство оказались слишком сложными
или не совсем подходящими для нашей цели.
JIRA уже проделывает большую тяжелую работу
с точки зрения i18n для клиентских ресурсов,
через свои преобразования web-resource transforms
и через доступность AJS.I18n.getText('some.key')


Поэтому мы выбрали простое решение.
Мы проедоставляем глобальный модуль i18Strings
в нашей web-pack конфигурации (смотрите первую статью из серии,
чтобы увидеть полный конфиг):

'''

externals: {
  i18nStrings: 'require("jira/nps/i18n")',
},

'''

Для этого требуется модуль, определенный 
стандартным JIRA web-resource
к которому применены преобразования (transforms).

Содержимое этого модуля:

'''
define("jira/nps/i18n", function () {
    var i18nPrefixes = $i18nPrefixes("survey.plugin");
    return extend(i18nPrefixes, {
        "common.words.save": AJS.I18n.getText("common.words.save"),
        //... keys excluded for brevity
        "user.picker.no.permission": AJS.I18n.getText("user.picker.no.permission")
    });
});

'''

$$i18nPrefixes - это специальная функция,
которая на самом деле преобразуется
нашим собственным преобразователем веб-ресурсов
в объект JSON,
содержащим все i18nized key → values 
для префикса 'survey.plugin'

Это всего лишь некоторый синтаксический сахар,
чтобы меньше нажимать на клавиши.
Этот подход должен быть реимплементирован
другим способом в дополнении atlassian-connect 
предназначенном для облака.

Затем, наконец, мы предоставляем модуль i18n
для всех наших компонентов React для импорта:

'''

import I18nHelper from './i18nHelper';

let i18nStrings = {};
try {
  i18nStrings = require('i18nStrings');
} catch (e) {
  // this is mainly here so that the unit tests compile. Otherwise it could be
  // an *import i18nStrings from "i18nStrings"*.
  // there doesn't seem to be a good way to 'inject' a global 'i18nStrings' via
  // mocha/babel when running tests
}

const i18n = new I18nHelper(i18nStrings);

export default i18n;
'''

Импортируемый нами I18Helper предоставляет тот же API интерфейс
getText(key, args)
что и AJS.I18n.getText()
Теперь в наших реакт-компонентах мы можем просто вызвать:


import i18n from './i18n';

const translatedText = i18n.getText('survey.plugin.title');


# Enter Redux Использование Redux

Ранее, при рассмотрении модульных тестов
мы обнаружили
какой-то a bit of a smell
в одном из наших компонентов - 
это был тот факт, что наша диаграмма
NPS dougnut была глубоко осведомлена о state
и выполняла REST запросы.
Это сделало тестирование менее чем идеальным,
а наш компонент - слишком сложным.
В идеале, в React большинство компонентов
должны быть просто "тупыми" функциями без сохранения состояния,
которые передают props
и ничего не знают о state.


Redux, это контейнер с предсказуемым state для JavaScript,
который призван решить эту проблему.
Объяснение Redux выходит далеко за рамки данной статьи
и не является необходимым,
поскольку документация Redux просто невероятна!
http://redux.js.org/

Давайте посмотрим, как использование Redux
может сделать еще одну сложную часть плагина NPS для JIRA
намного проще.
Взгляните пожалуйста на ConfigForm:

https://blog.developer.atlassian.com/wp-content/uploads/dac-import/config-form.png

Здесь есть несколько вещей, которыми нужно управлять:

- загрузка исходных данных для формы (issue types,
сами данные формы)
- сохранение и удаление
- ошибки, возвращаемые сервером при сохранении

Вот так выглядит компонент NPSAdmin React,
который отображает эту страницу:


'''
//imports excluded for brevity
const NPSAdmin = (props) => {
  if (props.deleteSurvey.deleted) {
    window.location.reload();
    return <div />;
  }

  if (props.configData.unexpectedError) {
    window.alert(i18n.getText('survey.plugin.unexpected.error'));
    window.location.reload();
    return <Spinner />;
  }

  if (props.configData.isLoading) {
    return <div className="admin-loading"><Spinner /></div>;
  }

  return (
    <ConfigForm
      baseUrl={props.configData.context.baseUrl}
      surveyInfo={props.configData.surveyInfo}
      issueTypes={props.configData.issueTypes}
      onSave={(survey) => props.handleSave(props.configData.context, survey)}
      onDelete={() => props.handleDelete(props.configData.context)}
      saveSuccess={props.saveSurvey.success}
      errorCollection={props.saveSurvey.errorCollection}
    />);
};
const mapStateToProps = (state) => ({ ...state });
const mapDispatchToProps = (dispatch) => ({
  handleDelete: (context) => {
    const shouldDelete = window.confirm(i18n.getText('survey.plugin.config.form.delete.survey.confirm'));
    if (shouldDelete) {
      dispatch(deleteSurvey(context));
    }
  },
  handleSave: (context, survey) => {
    dispatch(saveSurvey(context, survey));
  },
});

export default connect(mapStateToProps, mapDispatchToProps)(NPSAdmin);

'''

Это тупая функция без состояния (dumb stateless function)
Это позволяет очень легко проводить модульное тестирование:
мы можем просто пропускать это через различные props.
Не нужно иметь дело с асинхронными тестами и обратными
вызовами Promise,
как в нашем примере с NPS Donut chart.

Интересная часть, касающаяся Redux,
находится внизу.
Мы оборачиваем компонент NPSAdmin
с помощь вызова метода connect()
и передаем две mapping функции для mapping state to props
и для dispatching actions.

Функция mapStateToProps
вызывается при каждом изменении состояния
в нашем Redux store (хранилище)
mapDispatchToProps будет отправлять соответствующие actions
всякий раз,
когда ConfigForm запускает свои обработчики (handlers) onSave или OnDelete.

# Actions
Давайте кратко рассмотрим,
как выглядит действие сохранения (save action):

'''
//...other actions excluded for brevity
export const SURVEY_SAVE_ERRORS = 'SURVEY_SAVE_ERRORS';
export function surveySaveErrors(ex) {
  return (dispatch) => {
    if (ex.response && ex.response.status === 400) {
      return ex.response.json().then((errorCollection) =>
        dispatch({
          type: SURVEY_SAVE_ERRORS,
          errors: errorCollection,
        })
      );
    }

    dispatch(unexpectedError());
    return Promise.resolve();
  };
}

export function saveSurvey(context, newSurvey) {
  return (dispatch) =>

    fetch(`${encodeURI(context.baseUrl)}/rest/nps/1.0/${encodeURI(context.projectKey)}/surveys/${encodeURI(context.surveyId)}`, {
      method: 'PUT',
      headers: {
        Accept: 'application/json',
        'Content-Type': 'application/json',
      },
      credentials: 'same-origin',
      body: JSON.stringify(newSurvey),
    }).then(checkForError).then(() => {
      dispatch(surveySavedSuccessfully(newSurvey));
      dispatch(hideSavedSuccessfully());
    }).catch((ex) => {
      dispatch(surveySaveErrors(ex));
    });
}
'''

Теперь мы обрабатываем вызовы на сервере
и отправляем новые actions в зависимости от результата.
Тестировать это намного проще,
так как Redux продоставляет несколько helpers (redux-mock-store)


# Testing

Мы могли бы просто протестировать наш тупой компонент
NPSAdmin, предав различные props.
Однако мы можем провести и другой тест - чтобы убедиться,
что наш компонент NPSAdmin правильно реагирует на действия (actions),
dispatched (отправленные) в реальном магазине Redux:


'''

//imports excluded for brevity
function renderAdmin() {
  return renderStatefullyIntoDocument(<Provider store={store}><NPSAdmin /></Provider>);
}

let store;
let sandbox;
const context = { baseUrl: '/jira', projectKey: 'TEST', surveyId: 'sample-survey-id' };

describe('NPS Admin', () => {
  beforeEach(() => {
    store = createStore(adminApp);
    sandbox = sinon.sandbox.create();
  });

  //other tests excluded for brevity 
  it('is loading by default', () => {
    const el = renderAdmin();

    const titleField = TestUtils.scryRenderedDOMComponentsWithClass(el, 'title-field');
    expect(titleField.length).to.be(0);
    const spinner = TestUtils.scryRenderedDOMComponentsWithClass(el, 'admin-loading');
    expect(spinner.length).to.be(1);
  });

  it('renders config form when data received', () => {
    const el = renderAdmin();

    store.dispatch({
      type: RECEIVED_SURVEY_CONFIG,
      issueTypes: [],
      surveyAdmin: SAMPLE_SURVEY_RESP,
      context,
    });

    const titleField = TestUtils.scryRenderedDOMComponentsWithClass(el, 'title-field');
    expect(titleField.length).to.be(1);
  });
});

'''

Это гораздо лучший тест, чем тот, который мы видели ранее
с компонентом NPS Donut.


# That’s all for now folks Вот и все, ребята

Еще раз. Мы многое рассмотрели в этой части.
Мы рассмотрели анатомию нескольких различных компонентов React
и то, как возможность использования ES6
создает очень чистый и простой в поддержке код.
Мы также показали,
как протестировать и интернационализировать эти компоненты.
Наконец, мы смогли сделать сложную часть приложения намного
проще в обслуживании и модульном тестировании при помощи Redux!

Мы надеемся, что вам понравилась эта серия статей о том,
как писать плагины для JIRa,
используя при этом новый процесс разработки,
который обеспечивает работу с повторно используемыми 
компонентами пользовательского интерфейса,
как для сервера, так и для облака,
с использованием некоторых новейший инструментов, 
доступных сейчас в веб-разработке!

Мы уверены, что есть много предложений 
для улучшения этого подхода.
Пожалуйста, дайте нам знать об этом
в комментариях ниже!
