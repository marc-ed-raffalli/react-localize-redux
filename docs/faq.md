## Do I have to connect every component that needs translations?

No you shouldn't have to connect every component. To avoid this add translate to a parent component, and then pass translations 
down to stateless child components as props. See the [pass multiple translations to components](/features/#pass-multiple-translations-to-components) feature on one way to accomplish this.

## What if my translation data isn't in the required format?

If you don't have control over the translation data for your application you can use the [translationTransform](/api/action-creators/#initializelanguages-options) option. 
This allows you to write a function that takes in your custom translation data, and outputs the data in the required format.
See [Custom data format](/formatting-translation-data/#custom-data-format) for documentaion.

## How do I handle currency, date, and other localization transformations?

This logic is purposely excluded from react-localize-redux to ensure that both package size and API remian small. If you do require this logic you have the choice of writing it yourself, or using a third party library that specializes in that area e.g.([Moment](https://momentjs.com/) for dates).

Here's an example of a basic currency translation using [reselect](https://github.com/reactjs/reselect):

```javascript
import { initialize } from 'react-localize-redux';

const languages = ['en', 'fr'];

const translations = {
  currency: ["$ ${value}", "${value} $"]
};

store.dispatch(initialize(languages));
store.dispatch(addTranslation(translations));

const Transactions = ({translateCurrency}) => (
  <ul>
    <li>{translateCurrency(1000)}</li>     // renders $ 1,000 (en), 1 000 $ (fr)
    <li>{translateCurrency(100000)}</li>   // renders $ 100,000 (en), 100 000 $ (fr)
    <li>{translateCurrency(500)}</li>      // renders $ 500 (en), 500 $ (fr)
  </ul>
);

const getTranslateSelector = (state) => getTranslate(state.locale);
const getActiveLanguageSelector = (state) => getActiveLanguage(state.locale);

/**
 * Returns a function that takes a number, and formats it using
 * the current language, the currency translate data, and native toLocaleString 
 */
const getTranslateCurrency = createSelector(
  getTranslateSelector, getActiveLanguageSelector,
  (translate, language) => (value) => {
    const localizedValue = value.toLocaleString(language.code);
    return translate('currency', {value: localizedValue});
  }
);

const mapStateToProps = state => ({
  translateCurrency: getTranslateCurrency(state)
});

const LocalizedTransactions = connect(mapStateToProps)(Transactions);
```

## How does react-localize-redux differ from [react-intl](https://github.com/yahoo/react-intl)?

* **react-intl** is larger in size/complexity, and for good reason as it handles many things related to localization. e.g. Pluralization, currency. Where as with **react-localize-redux** you could still do pluralization, and currency, but you'd be writing the formatting functionality yourself. 
<br/>
<br/>
* **react-intl** doesn't work with Redux out of the box, and needs an additional library [react-intl-redux](https://github.com/ratson/react-intl-redux) to add support.
<br/>
<br/>
* For further discussion on this topic see [original github issue](https://github.com/ryandrewjohnson/react-localize-redux/issues/21).
