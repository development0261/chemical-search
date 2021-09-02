# Chemical Search

> A Vue.js App

## Build Setup

``` bash
# install dependencies
npm install

# serve with hot reload at localhost:8080
npm run dev
```

For detailed explanation on how things work, consult the [docs for vue-loader](http://vuejs.github.io/vue-loader).

## Features

- User can search text and they can includes operators(AND|OR|,(Comma)). Based on string and operators string converts into a words.
- With all words we get response from api as lists of Synonyms which user can see. If word is wrong then it will return nothing.
- Some of words are selected from list if list have same word when user have wrote before in string.
- When user click on submit they will see word(s) which they selected from the response list also user can update selected words and those words are converted into string which in special format as mentioned in documentation.
- User can clear the words with clear button.