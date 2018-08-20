# VueTagsBall

## Description
create a 3d tags ball for vue2

## Demo

[demo](https://test.fuckneusoft.com/)

## Install
```
npm install --save vue-tags-ball
```
## Usage
```javascript
//Gobal method
import TagsBall from 'vue-tags-ball'
Vue.use(TagsBall)

//Component method
import TagsBall from 'vue-tags-ball'
//...
components: {
    "tags-ball":TagsBall
  },
```
## Example
```html
<tags-ball v-bind:style='{"border":"2px solid black"}' :width='200' :tags='tags'/>
```
`tags` is a String[],like `{"tag1","tag2","tag3"}`.and you can bind your own style.

Warning: just watching the `tags` and `stop`,other props only init.

## Props
| Property | Type    | Description                    | Default       |
| -------- | ------- | ------------------------------ | ------------- |
| tags     | Array   | all the tag                    | []            |
| stop     | Boolean | stop the animation             | false         |
| width    | Number  | init the canvas widht          | 200           |
| height   | Number  | init the canvas height         | 200           |
| radius   | Number  | init the ball radius           | 100           |
| font     | String  | init the tag's label font      | "24px monaco" |
| fontMax  | Number  | init the tag's label max width | 60            |
| color    | String  | init the label's color         | "#fc8457"     |
