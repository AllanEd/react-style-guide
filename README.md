# Airbnb React/JSX Style Guide

*Ein erfahrungsgemäß angemessener Ansatz für React und JSX*

Dieser Style Guide basiert auf die aktuell vorherrschenden Javascript Standards, obwohl ein paar Konventionen (z.B. async/await oder statische Klassenvariablen) noch aufgenommen oder von Fall zu Fall verboten werden können. Derzeit ist alles, was vor der „stage 3“ liegt, weder in diesem Handbuch enthalten noch empfohlen.

## Inhaltsverzeichnis

  1. [Grundregeln](#grundregeln)
  1. [Class vs `React.createClass` vs zustandslos](#class-vs-reactcreateclass-vs-zustandslos)
  1. [Mixins](#mixins)
  1. [Namensgebung](#namensgebung)
  1. [Deklarierung](#deklarierung)
  1. [Ausrichtung](#ausrichtung)
  1. [Anführungszeichen](#anführungszeichen)
  1. [Abstände](#abstände)
  1. [Props](#props)
  1. [Refs](#refs)
  1. [Klammern](#klammern)
  1. [Tags](#tags)
  1. [Methoden](#methoden)
  1. [Anordnung](#anordnung)
  1. [`isMounted`](#ismounted)

## Grundregeln

  - Eine Datei darf nur eine React Komponente enthalten.
    - Es sind jedoch mehrere [zustandslose oder „pure“ Komponenten](https://facebook.github.io/react/docs/reusable-components.html#stateless-functions) pro Datei erlaubt. eslint: [`react/no-multi-comp`](https://github.com/yannickcr/eslint-plugin-react/blob/master/docs/rules/no-multi-comp.md#ignorestateless).
  - Verwende immer die JSX Syntax.
  - Verwende `React.createElement` nur, wenn die App nicht aus einer JSX Datei initialisiert wird.
  - [`react/forbid-prop-types`](https://github.com/yannickcr/eslint-plugin-react/blob/master/docs/rules/forbid-prop-types.md) erlaubt `arrays` und `objects` nur, wenn explizit angegeben wird was `arrays` und `objects` enthält, unter der Verwendung von `arrayOf`, `objectOf` oder `shape`.

## Class vs `React.createClass` vs zustandslos

  - Bei einem internen „state“ und/oder „refs“, ist `class extends React.Component`, `React.createClass` vorzuziehen. eslint: [`react/prefer-es6-class`](https://github.com/yannickcr/eslint-plugin-react/blob/master/docs/rules/prefer-es6-class.md) [`react/prefer-stateless-function`](https://github.com/yannickcr/eslint-plugin-react/blob/master/docs/rules/prefer-stateless-function.md)

    ```jsx
    // schlecht
    const Listing = React.createClass({
      // ...
      render() {
        return <div>{this.state.hello}</div>;
      }
    });

    // gut
    class Listing extends React.Component {
      // ...
      render() {
        return <div>{this.state.hello}</div>;
      }
    }
    ```

    Bei keinem „state“ oder „refs“, werden normale Funktionen (keine Pfeilfunktionen) gegenüber Klassen bevorzugt:

    ```jsx
    // schlecht
    class Listing extends React.Component {
      render() {
        return <div>{this.props.hello}</div>;
      }
    }

    // schlecht (es wird abgeraten sich auf die Schlussfolgerung des Funktionsnamens zu verlassen)
    const Listing = ({ hello }) => (
      <div>{hello}</div>
    );

    // gut
    function Listing({ hello }) {
      return <div>{hello}</div>;
    }
    ```

## Mixins

  - [Verwende keine Mixins](https://facebook.github.io/react/blog/2016/07/13/mixins-considered-harmful.html).

  > Warum? Mixins eröffnen implizite Abhängigkeiten, versuchsachen Namenskonflikte und eine beschleunigende Komplexität. Die meisten Anwendungsfälle für Mixins lassen sich besser über Komponenten, höhere Komponenten oder Utility-Module realisieren.

## Namensgebung

  - **Erweiterungen**: Verwende `.jsx` Erweiterungen für React Komponenten. eslint: [`react/jsx-filename-extension`](https://github.com/yannickcr/eslint-plugin-react/blob/master/docs/rules/jsx-filename-extension.md)
  - **Dateiname**: Verwende PascalCase für Dateinamen. Z.B., `ReservationCard.jsx`.
  - **Benennung der Referenzen**: Verwende PascalCase für React Komponenten und camelCase für seine Instanzen. eslint: [`react/jsx-pascal-case`](https://github.com/yannickcr/eslint-plugin-react/blob/master/docs/rules/jsx-pascal-case.md)

    ```jsx
    // schlecht
    import reservationCard from './ReservationCard';

    // gut
    import ReservationCard from './ReservationCard';

    // schlecht
    const ReservationItem = <ReservationCard />;

    // gut
    const reservationItem = <ReservationCard />;
    ```

  - **Benennung der Komponenten**: Verwende den Dateinamen als Komponentenname. Zum Beispiel, `ReservationCard.jsx` sollte folgenden Referenznamen haben `ReservationCard`. Allerdings sollte für eine Root Komponente eines Ordners `index.jsx` als Dateiname verwendet werden und der Ordnername als Komponentenname:

    ```jsx
    // schlecht
    import Footer from './Footer/Footer';

    // schlecht
    import Footer from './Footer/index';

    // gut
    import Footer from './Footer';
    ```
  - **Bennenung höherer Komponenten**: Verwende eine Komination vom Namen der höheren Komponente und den Namen der übergebenden Komponenten als den `displayName` der erzeugten Komponente. Zum Beispiel, wenn die höhere Komponente `withFoo()`, die Komponente `Bar` übergibt, sollte eine Komponente mit einem `displayName` von `withFoo(Bar)` erzeugen.

    > Warum? Der `displayName` einer Komponente könnte von Entwicklertools oder Fehlermeldungen verwendet werden und einen Wert zu haben, der klar ist und die Beziehung angibt hilft um zu verstehen was passiert.

    ```jsx
    // schlecht
    export default function withFoo(WrappedComponent) {
      return function WithFoo(props) {
        return <WrappedComponent {...props} foo />;
      }
    }

    // gut
    export default function withFoo(WrappedComponent) {
      function WithFoo(props) {
        return <WrappedComponent {...props} foo />;
      }

      const wrappedComponentName = WrappedComponent.displayName
        || WrappedComponent.name
        || 'Component';

      WithFoo.displayName = `withFoo(${wrappedComponentName})`;
      return WithFoo;
    }
    ```

  - **Bennenung der Props**: Vermeide die Verwendung von DOM Komponenten Prop-Namen für verschiedene Vorhaben.

    > Warum? Man erwartet, wie bei `style` und `className`, dass Props genau eine spezielle Sache bedeuten. Durch das Ändern dieser API für Untergruppen der App, wird der Code weniger lesbar und wartbar und kann Fehler verursachen.

    ```jsx
    // schlecht
    <MyComponent style="fancy" />

    // schlecht
    <MyComponent className="fancy" />

    // gut
    <MyComponent variant="fancy" />
    ```

## Deklarierung

  - Verwende nicht `displayName` für die Benennung von Komponenten. Benenne stattdessen die Komponente durch die Referenz.

    ```jsx
    // schlecht
    export default React.createClass({
      displayName: 'ReservationCard',
      // stuff goes here
    });

    // gut
    export default class ReservationCard extends React.Component {
    }
    ```

## Ausrichtung

  - Folge diese Ausrichtungsstil für die JSX Syntax. eslint: [`react/jsx-closing-bracket-location`](https://github.com/yannickcr/eslint-plugin-react/blob/master/docs/rules/jsx-closing-bracket-location.md) [`react/jsx-closing-tag-location`](https://github.com/yannickcr/eslint-plugin-react/blob/master/docs/rules/jsx-closing-tag-location.md)

    ```jsx
    // schlecht
    <Foo superLongParam="bar"
         anotherSuperLongParam="baz" />

    // gut
    <Foo
      superLongParam="bar"
      anotherSuperLongParam="baz"
    />

    // Wenn die Props auf eine Zeile passen, dann halte sie auf der gleichen Zeile.
    <Foo bar="bar" />

    // Kinder werden normal eingerückt
    <Foo
      superLongParam="bar"
      anotherSuperLongParam="baz"
    >
      <Quux />
    </Foo>

    // schlecht
    {showButton &&
      <Button />
    }

    // schlecht
    {
      showButton &&
        <Button />
    }

    // gut
    {showButton && (
      <Button />
    )}

    // gut
    {showButton && <Button />}
    ```

## Anführungszeichen

  - Verwende immer doppelte Anführungszeichen (`"`) für JSX Attribute aber einfache Anführungszeichen (`'`) für alles andere JS. eslint: [`jsx-quotes`](https://eslint.org/docs/rules/jsx-quotes)

    > Warum? Normale HTML Attribute verwenden typischerweise auch doppelte Anführungszeichen anstatt einfache und JSX spiegelt diese Konvention wieder.

    ```jsx
    // schlecht
    <Foo bar='bar' />

    // gut
    <Foo bar="bar" />

    // schlecht
    <Foo style={{ left: "20px" }} />

    // gut
    <Foo style={{ left: '20px' }} />
    ```

## Abstände

  - Füge immer ein einzelnes Leerzeichen in selbstschließendes Tag ein. eslint: [`no-multi-spaces`](https://eslint.org/docs/rules/no-multi-spaces), [`react/jsx-tag-spacing`](https://github.com/yannickcr/eslint-plugin-react/blob/master/docs/rules/jsx-tag-spacing.md)

    ```jsx
    // schlecht
    <Foo/>

    // sehr schlecht
    <Foo                 />

    // schlecht
    <Foo
     />

    // gut
    <Foo />
    ```

  - Fülle die geschweiften JSX-Klammern nicht mit Zwischenräumen. eslint: [`react/jsx-curly-spacing`](https://github.com/yannickcr/eslint-plugin-react/blob/master/docs/rules/jsx-curly-spacing.md)

    ```jsx
    // schlecht
    <Foo bar={ baz } />

    // gut
    <Foo bar={baz} />
    ```

## Props

  - Benutze immer camelCase für Props Namen.

    ```jsx
    // schlecht
    <Foo
      UserName="hello"
      phone_number={12345678}
    />

    // gut
    <Foo
      userName="hello"
      phoneNumber={12345678}
    />
    ```

  - Lasse den Wert der Prop weg, wenn er immer explizit `true` ist. eslint: [`react/jsx-boolean-value`](https://github.com/yannickcr/eslint-plugin-react/blob/master/docs/rules/jsx-boolean-value.md)

    ```jsx
    // schlecht
    <Foo
      hidden={true}
    />

    // gut
    <Foo
      hidden
    />

    // gut
    <Foo hidden />
    ```

  - Gebe immer ein `alt` Prop an `<img>` Tags an. Wenn das Bild repräsentativ ist, kann `alt` ein leerer String sein oder der `<img>` Tag muss `role="presentation"` haben. eslint: [`jsx-a11y/alt-text`](https://github.com/evcohen/eslint-plugin-jsx-a11y/blob/master/docs/rules/alt-text.md)

    ```jsx
    // schlecht
    <img src="hello.jpg" />

    // gut
    <img src="hello.jpg" alt="Me waving hello" />

    // gut
    <img src="hello.jpg" alt="" />

    // gut
    <img src="hello.jpg" role="presentation" />
    ```

  - Verwende keine Wörter wie "image", "photo" oder "picture" in `<img>` `alt` Props. eslint: [`jsx-a11y/img-redundant-alt`](https://github.com/evcohen/eslint-plugin-jsx-a11y/blob/master/docs/rules/img-redundant-alt.md)

    > Warum? Screenreader geben `img` Elemente bereits als Bilder an, darum muss man diese Information nicht im alt Text mit angeben.

    ```jsx
    // schlecht
    <img src="hello.jpg" alt="Picture of me waving hello" />

    // gut
    <img src="hello.jpg" alt="Me waving hello" />
    ```

  - Verwende nur gültige, keine abstrakten [ARIA Regeln](https://www.w3.org/TR/wai-aria/#usage_intro). eslint: [`jsx-a11y/aria-role`](https://github.com/evcohen/eslint-plugin-jsx-a11y/blob/master/docs/rules/aria-role.md)

    ```jsx
    // schlecht - keine ARIA Regel
    <div role="datepicker" />

    // schlecht - abstrakte ARIA Regel
    <div role="range" />

    // gut
    <div role="button" />
    ```

  - Verwende nicht `accessKey` an Elementen. eslint: [`jsx-a11y/no-access-key`](https://github.com/evcohen/eslint-plugin-jsx-a11y/blob/master/docs/rules/no-access-key.md)

  > Warum? Inkonsistenzen zwischen Tastaturkürzeln und Tastaturbefehlen, die von Personen verwendet werden, die Screenreader und Tastaturen verwenden, erschweren die Zugänglichkeit.

  ```jsx
  // schlecht
  <div accessKey="h" />

  // gut
  <div />
  ```

  - Vermeide die Verwendung eines Array Indizes als `key` Prop, bevorzuge eine festen ID. eslint: [`react/no-array-index-key`](https://github.com/yannickcr/eslint-plugin-react/blob/master/docs/rules/no-array-index-key.md)

> Warum? Keine feste ID zu verwenden [ist ein Anit-Pattern](https://medium.com/@robinpokorny/index-as-a-key-is-an-anti-pattern-e0349aece318) weil es die Leistung negativ beeinflussen und Probleme mit dem Komponentenzustand verursachen kann.

Wir empfehlen nicht, Indizes für Schlüssel zu verwenden, wenn sich die Reihenfolge der Elemente ändern kann.

  ```jsx
  // schlecht
  {todos.map((todo, index) =>
    <Todo
      {...todo}
      key={index}
    />
  )}

  // gut
  {todos.map(todo => (
    <Todo
      {...todo}
      key={todo.id}
    />
  ))}
  ```

  - Definiere immer explizite Standard-Props für alle nicht benötigten Props.

  > Warum? propTypes sind eine Form der Dokumentation und durch das Anbieten von defaultProps muss der Leser des Codes keine großen Annahmen machen. Zusätzlich  are a form of documentation, and providing defaultProps means the reader of your code doesn’t have to assume as much. Darüber hinaus kann es bedeuten, dass der Code bestimmte Typenprüfungen weglassen kann.
  
  ```jsx
  // schlecht
  function SFC({ foo, bar, children }) {
    return <div>{foo}{bar}{children}</div>;
  }
  SFC.propTypes = {
    foo: PropTypes.number.isRequired,
    bar: PropTypes.string,
    children: PropTypes.node,
  };

  // gut
  function SFC({ foo, bar, children }) {
    return <div>{foo}{bar}{children}</div>;
  }
  SFC.propTypes = {
    foo: PropTypes.number.isRequired,
    bar: PropTypes.string,
    children: PropTypes.node,
  };
  SFC.defaultProps = {
    bar: '',
    children: null,
  };
  ```

  - Verwende verteilte Props sparsam.
  > Warum? Andernfalls ist es wahrscheinlicher, dass unnötige Props an Komponenten weitergeben. Und für React v15.6.1 und älter, kannst du [ungültige HTML Attribute an den DOM weitergeben](https://reactjs.org/blog/2017/09/08/dom-attributes-in-react-16.html).

  Ausnahmen:

  - Höhere Komponenten (HOC) die Proxy-Props weitergeben und propTypes hochziehen

  ```jsx
  function HOC(WrappedComponent) {
    return class Proxy extends React.Component {
      Proxy.propTypes = {
        text: PropTypes.string,
        isLoading: PropTypes.bool
      };

      render() {
        return <WrappedComponent {...this.props} />
      }
    }
  }
  ```

  - Ausbreiten von Objekten mit bekannten, expliziten Props. Dies kann besonders nützlich sein, wenn React-Komponenten mit Mochas beforeEach-Konstrukt getestet werden.

  ```jsx
  export default function Foo {
    const props = {
      text: '',
      isPublished: false
    }

    return (<div {...props} />);
  }
  ```

  Hinweise zur Verwendung:
  Filter überflüssig Props wenn möglich heraus. Verwenden auch [prop-types-exact](https://www.npmjs.com/package/prop-types-exact) um Bugs zu verhindern.

  ```jsx
  // schlecht
  render() {
    const { irrelevantProp, ...relevantProps  } = this.props;
    return <WrappedComponent {...this.props} />
  }

  // gut
  render() {
    const { irrelevantProp, ...relevantProps  } = this.props;
    return <WrappedComponent {...relevantProps} />
  }
  ```

## Refs

  - Verwende immer Ref Callbacks. eslint: [`react/no-string-refs`](https://github.com/yannickcr/eslint-plugin-react/blob/master/docs/rules/no-string-refs.md)

    ```jsx
    // schlecht
    <Foo
      ref="myRef"
    />

    // gut
    <Foo
      ref={(ref) => { this.myRef = ref; }}
    />
    ```

## Klammern

  - Umschließe JSX Tags in Klammern, wenn sie mehr als eine Zeile umfassen. eslint: [`react/jsx-wrap-multilines`](https://github.com/yannickcr/eslint-plugin-react/blob/master/docs/rules/jsx-wrap-multilines.md)

    ```jsx
    // schlecht
    render() {
      return <MyComponent variant="long body" foo="bar">
               <MyChild />
             </MyComponent>;
    }

    // gut
    render() {
      return (
        <MyComponent variant="long body" foo="bar">
          <MyChild />
        </MyComponent>
      );
    }

    // gut, wenn einzeilig
    render() {
      const body = <div>hello</div>;
      return <MyComponent>{body}</MyComponent>;
    }
    ```

## Tags

  - Bei keinen Kindern, immer selbstschließende Tags. eslint: [`react/self-closing-comp`](https://github.com/yannickcr/eslint-plugin-react/blob/master/docs/rules/self-closing-comp.md)

    ```jsx
    // schlecht
    <Foo variant="stuff"></Foo>

    // gut
    <Foo variant="stuff" />
    ```

  - Hat die Komponente mehrzeilige Eigenschaften, schließe den Tag auf einer neuen Zeile. eslint: [`react/jsx-closing-bracket-location`](https://github.com/yannickcr/eslint-plugin-react/blob/master/docs/rules/jsx-closing-bracket-location.md)

    ```jsx
    // schlecht
    <Foo
      bar="bar"
      baz="baz" />

    // gut
    <Foo
      bar="bar"
      baz="baz"
    />
    ```

## Methoden

  - Verwende Pfeilfunktionen, um lokale Variablen zu schließen.

    ```jsx
    function ItemList(props) {
      return (
        <ul>
          {props.items.map((item, index) => (
            <Item
              key={item.key}
              onClick={() => doSomethingWith(item.name, index)}
            />
          ))}
        </ul>
      );
    }
    ```

  - Binde Eventhandler für die Rendermethode im Konstruktor. eslint: [`react/jsx-no-bind`](https://github.com/yannickcr/eslint-plugin-react/blob/master/docs/rules/jsx-no-bind.md)

    > Warum? Ein bind-Aufruf in der Render-Methode erstellt bei jedem Aufruf von Render eine neue Funktion.

    ```jsx
    // schlecht
    class extends React.Component {
      onClickDiv() {
        // do stuff
      }

      render() {
        return <div onClick={this.onClickDiv.bind(this)} />;
      }
    }

    // gut
    class extends React.Component {
      constructor(props) {
        super(props);

        this.onClickDiv = this.onClickDiv.bind(this);
      }

      onClickDiv() {
        // do stuff
      }

      render() {
        return <div onClick={this.onClickDiv} />;
      }
    }
    ```

  - Verwende kein Unterstrich-Präfix für interne Methoden einer React-Komponente.
    > Warum? Unterstich-Präfixe werden in manchen Sprachen als Konvention verwendet um etwas als privat zu kennzeichnen. Aber anders als in diesen Sprachen unterstützt JavaScript die Kennzeichnung von privat nicht nativ, alles ist öffentlich. Unabhängig von den Absichten macht das Hinzufügen eines Unterstich-Präfix die Eigenschaft nicht privat und jede Eigenschaft (mit Unterstrich-Präfix oder nicht) sollte als öffentlich behandelt werden. Siehe das folgende Problem [#1024](https://github.com/airbnb/javascript/issues/1024), and [#490](https://github.com/airbnb/javascript/issues/490) für eine tiefere Diskussion.

    ```jsx
    // schlecht
    React.createClass({
      _onClickSubmit() {
        // mache bestimmte Dinge
      },

      // mache bestimmte Dinge
    });

    // gut
    class extends React.Component {
      onClickSubmit() {
        // mache bestimmte Dinge
      }

      // mache andere Dinge
    }
    ```

  - Stelle sicher, dass ein Wert in der `render` Methode zurückgegeben wird. eslint: [`react/require-render-return`](https://github.com/yannickcr/eslint-plugin-react/blob/master/docs/rules/require-render-return.md)

    ```jsx
    // schlecht
    render() {
      (<div />);
    }

    // gut
    render() {
      return (<div />);
    }
    ```

## Anordnung

  - Anordnung für `class extends React.Component`:

  1. Optionale `static` Methoden
  1. `constructor`
  1. `getChildContext`
  1. `componentWillMount`
  1. `componentDidMount`
  1. `componentWillReceiveProps`
  1. `shouldComponentUpdate`
  1. `componentWillUpdate`
  1. `componentDidUpdate`
  1. `componentWillUnmount`
  1. *clickHandlers oder eventHandlers* wie `onClickSubmit()` oder `onChangeDescription()`
  1. *getter Methoden für `render`* wie `getSelectReason()` oder `getFooterContent()`
  1. *Optionale render Methods* wie `renderNavigation()` oder `renderProfilePicture()`
  1. `render`

  - Definition von `propTypes`, `defaultProps`, `contextTypes`, etc...

    ```jsx
    import React from 'react';
    import PropTypes from 'prop-types';

    const propTypes = {
      id: PropTypes.number.isRequired,
      url: PropTypes.string.isRequired,
      text: PropTypes.string,
    };

    const defaultProps = {
      text: 'Hello World',
    };

    class Link extends React.Component {
      static methodsAreOk() {
        return true;
      }

      render() {
        return <a href={this.props.url} data-id={this.props.id}>{this.props.text}</a>;
      }
    }

    Link.propTypes = propTypes;
    Link.defaultProps = defaultProps;

    export default Link;
    ```

  - Anordnung von `React.createClass`: eslint: [`react/sort-comp`](https://github.com/yannickcr/eslint-plugin-react/blob/master/docs/rules/sort-comp.md)

  1. `displayName`
  1. `propTypes`
  1. `contextTypes`
  1. `childContextTypes`
  1. `mixins`
  1. `statics`
  1. `defaultProps`
  1. `getDefaultProps`
  1. `getInitialState`
  1. `getChildContext`
  1. `componentWillMount`
  1. `componentDidMount`
  1. `componentWillReceiveProps`
  1. `shouldComponentUpdate`
  1. `componentWillUpdate`
  1. `componentDidUpdate`
  1. `componentWillUnmount`
  1. *clickHandlers oder eventHandlers* wie `onClickSubmit()` oder `onChangeDescription()`
  1. *getter Methods für `render`* wie `getSelectReason()` oder `getFooterContent()`
  1. *Optionale render Methoden* wie `renderNavigation()` oder `renderProfilePicture()`
  1. `render`

## `isMounted`

  - Verwende nicht `isMounted`. eslint: [`react/no-is-mounted`](https://github.com/yannickcr/eslint-plugin-react/blob/master/docs/rules/no-is-mounted.md)

  > Warum? [`isMounted` ist ein Anti-Pattern][anti-pattern], es ist unter ES6 Klassen nicht verfügbar und es ist auf dem Weg offiziell als veraltet zu gelten.

  [Anti-Pattern]: https://facebook.github.io/react/blog/2015/12/16/ismounted-antipattern.html

## Übersetzungen

  Dieser JSX/React Style Guide ist auch in anderen Sprachen verfügbar:

  - ![cn](https://raw.githubusercontent.com/gosquared/flags/master/flags/flags/shiny/24/China.png) **Chinese (Simplified)**: [JasonBoy/javascript](https://github.com/JasonBoy/javascript/tree/master/react)
  - ![tw](https://raw.githubusercontent.com/gosquared/flags/master/flags/flags/shiny/24/Taiwan.png) **Chinese (Traditional)**: [jigsawye/javascript](https://github.com/jigsawye/javascript/tree/master/react)
  - ![gb](https://raw.githubusercontent.com/gosquared/flags/master/flags/flags/shiny/24/United-Kingdom.png) **English**: [airbnb/javascript/react](https://github.com/airbnb/javascript/tree/master/react)
  - ![es](https://raw.githubusercontent.com/gosquared/flags/master/flags/flags/shiny/24/Spain.png) **Español**: [agrcrobles/javascript](https://github.com/agrcrobles/javascript/tree/master/react)
  - ![jp](https://raw.githubusercontent.com/gosquared/flags/master/flags/flags/shiny/24/Japan.png) **Japanese**: [mitsuruog/javascript-style-guide](https://github.com/mitsuruog/javascript-style-guide/tree/master/react)
  - ![kr](https://raw.githubusercontent.com/gosquared/flags/master/flags/flags/shiny/24/South-Korea.png) **Korean**: [apple77y/javascript](https://github.com/apple77y/javascript/tree/master/react)
  - ![pl](https://raw.githubusercontent.com/gosquared/flags/master/flags/flags/shiny/24/Poland.png) **Polish**: [pietraszekl/javascript](https://github.com/pietraszekl/javascript/tree/master/react)
  - ![Br](https://raw.githubusercontent.com/gosquared/flags/master/flags/flags/shiny/24/Brazil.png) **Portuguese**: [ronal2do/javascript](https://github.com/ronal2do/airbnb-react-styleguide)
  - ![ru](https://raw.githubusercontent.com/gosquared/flags/master/flags/flags/shiny/24/Russia.png) **Russian**: [leonidlebedev/javascript-airbnb](https://github.com/leonidlebedev/javascript-airbnb/tree/master/react)
  - ![th](https://raw.githubusercontent.com/gosquared/flags/master/flags/flags/shiny/24/Thailand.png) **Thai**: [lvarayut/javascript-style-guide](https://github.com/lvarayut/javascript-style-guide/tree/master/react)
  - ![tr](https://raw.githubusercontent.com/gosquared/flags/master/flags/flags/shiny/24/Turkey.png) **Turkish**: [alioguzhan/react-style-guide](https://github.com/alioguzhan/react-style-guide)
  - ![ua](https://raw.githubusercontent.com/gosquared/flags/master/flags/flags/shiny/24/Ukraine.png) **Ukrainian**: [ivanzusko/javascript](https://github.com/ivanzusko/javascript/tree/master/react)
  - ![vn](https://raw.githubusercontent.com/gosquared/flags/master/flags/flags/shiny/24/Vietnam.png) **Vietnam**: [uetcodecamp/jsx-style-guide](https://github.com/UETCodeCamp/jsx-style-guide)

**[⬆ zurück zum Anfang](#inhaltsverzeichnis)**
