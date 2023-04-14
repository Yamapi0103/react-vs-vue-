# react vs vue 用法上的差別

# Pass props

## 驗證props

react: prop-types、typescript

```jsx
// prop-types

import React, { Component } from 'react';
import PropTypes from 'prop-types';

class MyComponent extends Component {
  // ...
}

MyComponent.propTypes = {
  name: PropTypes.string.isRequired,
  age: PropTypes.number,
  isStudent: PropTypes.bool
};
```

```jsx
// typescript

interface Props {
name: string;
age?: number;
isStudent?: boolean;
}

const MyComponent(props: Props){
...
}
```

vue: 內建型別驗證

```jsx
<template>
  <!-- ... -->
</template>

<script>
export default {
  props: {
    name: {
      type: String,
      required: true
    },
    age: {
      type: Number,
      default: 18,
			validator(value){
				return value<=20
			}
    },
    isStudent: {
      type: Boolean,
      default: true
    },
  },
  // ...
}
</script>
```

- Boolean type props will be false by default ⇒ 不用特別定義 default: false
- required 和 default 不會同時存在（雖然不會噴錯）
- default 是當外面根本沒傳這個props（undefined）時會走的路，所以外面假如設一個props為null 是不會走default的囉～

## 傳入一個物件

```jsx
// in react

// 父元件 (Parent.js)

const Parent = () => {
	const user = {
	  name: 'John Doe',
	  age: 30
	};
  return (
    <Child name={user.name} age={user.age} />
  )
}

// 子元件 (Child.js)
const Child = (props) => {
  const { name, age } = props;
  return (
    <div>
      <p>Name: {name}</p>
      <p>Age: {age}</p>
    </div>
  )
}
```

vue使用v-bind綁定一個變數

可用冒號作為簡寫

```jsx
// in vue

// 父元件 (Parent.vue)
<template>
    <child :name="user.name" :age="user.age" />
</template>

<script>
export default {
  data() {
    return {
			user:{
	      name: 'John Doe',
	      age: 30 			
			}
    }
  }
}
</script>

// 子元件 (Child.vue)
<template>
  <div>
    <p>Name: {{ name }}</p>
    <p>Age: {{ age }}</p>
  </div>
</template>

<script>
export default {
  props: {
		name:String,
		age: Number
  }
}
</script>
```

## 展開怎麼傳？

```jsx
// in react 
// {...yourObject}

// 父元件 (Parent.js)

const Parent = () => {
	const user = {
	  name: 'John Doe',
	  age: 30
	};
  return (
    <Child {...user} />
  )
}

```

**`[v-bind` without an argument](https://vuejs.org/guide/essentials/template-syntax.html#dynamically-binding-multiple-attributes)**
 binds all the properties of an object as attributes of the target element.

```jsx
// in vue
// pass **v-bind**={yourObject} 

// 父元件 (Parent.vue)
<template>
    <child v-bind="user" />
</template>

```

## 拿attributes props(eg: class, style)

沒什麼好多說的，react 一樣用props接

```jsx
// in react
// 父元件 (Parent.js)

const Parent = () => {
	const user = {
	  name: 'John Doe',
	  age: 30,
	};
  return (
    <Child {...user} className="www" id="xxx" />
  )
}

// 子元件 (Child.js)
const Child = (props) => {
  const { name, age, className, id } = props;
	return (
    <div id={id} className={className}>
      <p>Name: {name}</p>
      <p>Age: {age}</p>
    </div>
  )
}
```

vue 用 $attrs 可拿到 除props、class、style 以外的屬性，不過vue 3將props以外的屬性都包在$attrs

[$attrs includes class & style | Vue 3 Migration Guide](https://v3-migration.vuejs.org/breaking-changes/attrs-includes-class-style.html)

inheritAttrs：默认值 true，继承所有的父组件属性，作为普通的HTML特性应用在子组件的根元素上，如果你不希望组件的根元素继承特性设置 **inheritAttrs: false**

```jsx
// in vue

// 父元件 (Parent.vue)
<template>
    <child v-bind="user" class="www" :style={color:'red'} id="xxx" data-a="a"/>
</template>

<script>
export default {
  data() {
    return {
      user: {
        name: 'John Doe',
        age: 30,
      }
    }
  }
}
</script>

// 子元件 (Child.vue)

props:{name,age}

子組件會render成：
（效果相當於v-bind="$attrs"）
  <div class="www" :style={color:'red'} id="xxx" data-a="a"/>
    <p>Name: {{ name }}</p>
    <p>Age: {{ age }}</p>
  </div>

```

# 數據綁定：Vue採用雙向數據綁定，React採用單向數據流。

react 須自行傳入onChange callback

```jsx
// react

import React, { useState } from 'react';

const InputComponent = () => {
  const [message, setMessage] = useState('');

  const handleChange = (event) => {
    setMessage(event.target.value);
  }

  return (
    <input type="text" value={message} onChange={handleChange} />
  );
}

export default InputComponent;
```

vue v-model

```jsx
// in vue

<template>
  <div>
    <input type="text" v-model="message" />
    <p>{{ message }}</p>
  </div>
</template>

<script>
export default {
  data() {
    return {
      message: 'Hello World'
    }
  }
}
</script>
```

延伸：v-model 相當於

```jsx
<input type="text"
  v-bind:value="message" 
  v-on:input="message = $event.target.value" />
```

vue 組件上也可以使用v-model

```jsx
// 父
<template>
  <div>
    <custom-input v-model="message"></custom-input>
    <p>{{ message }}</p>
  </div>
</template>

<script>
import CustomInput from './CustomInput.vue'
export default {
	components: {CustomInput},
  data() {
    return {
      message: 'Hello World'
    }
  }
}
</script>

// 子 CustomInput.vue
<template>
  <div>
    <input type="text" 
       v-bind:value="value" 
       v-on:input="$emit('input', $event.target.value)"/>
  </div>
</template>
<script>
export default {
  props: ['value']
}
</script>
```

如果子組件不想用value當props 名稱的話，可以自訂props name

```jsx
<template>
  <div>
    <input
      type="text"
      :value="message"
      v-on:input="$emit('input', $event.target.value)"
    />
  </div>
</template>

<script>
export default {
  model: {
    prop: "message",
    // event: "change" // 連input都可改成change
  },
  props: ["message"]
};
</script>
```

# 插槽

## 傳遞標籤之間的內容

react: props.children

vue: slot

```jsx
// in vue

// 父
<my-component>
  <p>This content will be rendered in the slot of my-component</p>
</my-component>

// 子
<template>
  <div>
    <h1>My Component</h1>
    <slot></slot>
  </div>
</template>
```

## 指定插入位置

```haskell
// in react
// 子 
const MyComponent = ({header,children,footer}) => {
  return (
    <div>
      <header>
        {header}
      </header>
      <main>
        {children}
      </main>
      <footer>
        {footer}
      </footer>
    </div>
  );
};

// 父

<MyComponent header={<h1>This is the header</h1>} footer={<h1>This is the footer</h1>}>
  <p>This content will be rendered in the main of MyComponent</p>
</MyComponent>
```

```jsx
// 若不想將children直接當props傳入
// 可改用children方式傳入
// 

// 子
const Header = (props) => <header>{props.children}</header>;
const Body = (props) => <main>{props.children}</main>;
const Footer = (props) => <footer>{props.children}</footer>;

const MyComponent = ({ children }) => {
  const childs = React.Children.toArray(children);
  const header = childs.find((el) => el.type === Header);
  const body = childs.find((el) => el.type === Body);
  const footer = childs.find((el) => el.type === Footer);

  return (
    <div className="container">
      {!!header && header}
      {!!body && body}
      {!!footer && footer}
    </div>
  );
};

MyComponent.Header = Header;
MyComponent.Body = Body;
MyComponent.Footer = Footer;
export default MyComponent

--------------------------------

// 父
<MyComponent>
  <MyComponent.Footer>
    <h1> this is footer </h1>
  </MyComponent.Footer>
  <MyComponent.Header>
    <h1> this is header </h1>
  </MyComponent.Header>
  <MyComponent.Body>
		<div>content...</div>
  </MyComponent.Body>
</MyComponent>

// 因為子元件有針對傳入children type依序插入對的位置
// (而不是直接render props.children)
// 所以外層只要有指定好元件名稱，即使亂放位置也會放入對應的位置
// 以上例會是：
<div class="container">
	<header>
	  <h1> this is header </h1>
	</header>
  <main>
	  <div> content... </div>
	</main>
  <footer>
	  <h1> this is footer </h1>
	</footer>
</div>
```

```jsx
// in vue

// 子

<template>
  <div>
    <header>
      <slot name="header"></slot>
    </header>
    <main>
      <slot></slot>
    </main>
    <footer>
      <slot name="footer"></slot>
    </footer>
  </div>
</template>

// 父

<my-component>
  <template v-slot:header>
    <h1>This is the header</h1>
  </template>
  <p>This content will be rendered in the default slot of my-component</p>
  <template v-slot:footer>
    <h1>This is the footer</h1>
  </template>
</my-component>

// v-slot 可用'#'簡寫
/*  eg: 
  <template #header>
    <h1>This is the header</h1>
  </template>
*/ 
```

- vue 其實是用$slots接到外部傳進來的內容(類似react children)，但他提供外部對slot命名以插入對應的位置。
- In React, you can specify the placement of "props children" by wrapping them in custom HTML elements or components within the parent component's JSX.(像左例). So we can create slots based on the type

## 插槽內容子傳父

相關應用範例： 進去調成手機版後可發現每塊swiper可封裝組件，滑動UI塊可給外層實作

[王者荣耀官网-腾讯游戏](https://pvp.qq.com/m/)

參考教學：

[](https://www.bilibili.com/video/BV1S4411W79F?p=11&vd_source=b7b95c88e0694fa186b56e535ce87b9d)

 React: render-props

```jsx
// in react

// 由於本人比較少使用react render-props
// 拿右邊用chatGPT翻成react大概會長這樣
// 還沒實際測過能否work

const SwiperContainer = (props) => {
  const {categories} = props
  const [active, setActive] = useState(0);

  return (
    <div className="card-body pt-3">
      <div className="nav jc-between">
        {categories.map((category, i) => (
          <div
            className={`nav-item ${active === i ? 'active' : ''}`}
            key={i}
            onClick={() => setActive(i)}
          >
            <div className="nav-link">{category.name}</div>
          </div>
        ))}
      </div>
      <div className="pt-3">
        <Swiper ref={(swiper) => {
          if (swiper) {
            swiper.slideTo(active);
          }
        }}>	
          {categories.map((category, i)=>{
						<SwiperSlide key={i}>
							{ props.render(category) }
					 </SwiperSlide>
					})} 
        </Swiper>
      </div>
    </div>
  );
};

export default SwiperContainer;

const Home = (props) => {
	return (
		<>
			<h1>新聞資訊</h1>
			<SwiperContainer
				categories={newsCats}
				render={(category)=>(
					// 拿到category.newsList實作新聞資訊的UI
				)}
			/>
      <h1>英雄列表</h1>

			<SwiperContainer
				categories={heroCats}
				render={(category)=>(
					// 拿到category.heroList實作英雄列表的UI
				)}
			/>
    </>
	)
}
```

SwiperContainer 只需要管swiper部分就好，畫面呈現交給外部去實作

 Vue: scoped slots

```jsx
// in vue

// SwiperContainer.vue
<template>
    <div class="card-body pt-3">
      <div class="nav jc-between">
        <div
          class="nav-item"
          :class="{active: active===i}"
          v-for="(category, i) in categories"
          :key="i"
          @click="$refs.list.swiper.slideTo(i)"
         >
         <div class="nav-link">{{category.name}}</div>
        </div>
      </div>
      <div class="pt-3">
        <swiper ref="list" :options="{autoHeight: true}" @slide-change="()=>active=$refs.list.swiper.realIndex">
          <swiper-slide v-for="(category, i) in categories" :key="i">
            <slot name="items" :category="category"></slot>
          </swiper-slide>
        </swiper>
      </div>
    </div>
</template>

<script>
  export default {
    props: {
      categories: { type: Array, required: true }
    },
    data() {
      return {
        active: 0
      };
    }
  };
</script>

// 外層拿到子層傳出來的資料，自己實作滑動畫面ＵＩ
// Home.vue
<SwiperContainer :categories="newsCats">
	<h1>新聞資訊</h1>
  <template #items="{category}">
    <router-link
      tag="div"
      :to="`articles/${news._id}`"
      class="py-2 fs-lg d-flex"
      v-for="(news,i) in category.newsList"
      :key="i"
    >
      <span class="text-info">[{{news.categoryName}}]</span>
      <span class="px-2">|</span>
      <span class="flex-1 text-dark-1 text-ellipsis pr-2">{{news.title}}</span>
      <span class="text-grey fs-sm">{{news.createdAt | date}}</span>
    </router-link>
  </template>
</SwiperContainer>
<SwiperContainer :categories="heroCats">
  <h1>英雄列表</h1>
  <template #items="{category}">
    <div class="d-flex flex-wrap" style="margin: 0 -0.5rem;">
      <router-link
        tag="div"
        :to="`/heroes/${hero._id}`"
        class="p-2 text-center"
        style="width: 20%;"
        v-for="(hero, i) in category.heroList"
        :key="i"
      >
        <img :src="hero.avatar" class="w-100" />
        <div>{{hero.name}}</div>
      </router-link>
    </div>
  </template>
</SwiperContainer>
```

# CSS-in-JS

## 常用的工具比較

工具react 通常使用 styled-component

```jsx
	// in react

import styled from 'styled-components';

const MyButton = styled.button`
  background: red;
  color: white;
  padding: 10px 20px;
  border-radius: 4px;
  cursor: pointer;
`;

function App() {
  return <MyButton>Click me</MyButton>;
}
```

vue 本身有提供 scoped

```jsx
// in vue
<template>
  <div>
    <button class="my-button">Click me</button>
  </div>
</template>

<style **scoped**>
.my-button {
  background: red;
  color: white;
  padding: 10px 20px;
  border-radius: 4px;
  cursor: pointer;
}
</style>
```

## 父層影響組件內部樣式

styled-components 可覆寫組件內部樣式

```jsx
// in react

import styled from 'styled-components';

const StyledModal = styled(PitayaFormModal)`
  .formModal{
    width: 55vw;
	}
	.formModal-body {
		h1 {
			color: red;
		}
	}
`;

function App() {
  return (
    <StyledModal>
			...
    </StyledModal>
}
```

vue可用::v-deep 影響到組件內部樣式

```jsx
// in vue

<template>
  <div id="app">
		<StyledModal>
			...
    </StyledModal>
	</div>
</template>
...
<style scoped>
  ::v-deep {
    .formModal {
	    width: 55vw;
		}
    .formModal-body {
		  h1 {
			  color: red;
		  }
	  }
	}
</style>
```

# **訪問組件或DOM元素**

 react 

- class-component 可以使用"createRef"
- funciton-component可用"useRef"來創建一個 ref

```jsx
// in react 

import React, { useRef } from 'react';

function InputExample() {
  const inputRef = useRef(null);

  const showMessage = () => {
    console.log(inputRef.current.value);
  };

  return (
    <div>
      <input ref={inputRef} />
      <button onClick={showMessage}>Show Message</button>
    </div>
  );
}

export default InputExample;

```

vue 可用"ref”訪問

```jsx
// in vue

<template>
  <div>
    <input ref="input" v-model="message" />
    <button @click="showMessage">Show Message</button>
  </div>
</template>

<script>
export default {
  data() {
    return {
      message: ''
    }
  },
  methods: {
    showMessage() {
      console.log(this.$refs.input.value)
    }
  }
}
</script>
```

# 測驗

[https://presenter.ahaslides.com/apps/presentations](https://presenter.ahaslides.com/apps/presentations)

# 參考資料

- v-bind 直接將物件展開傳入

[2-2 元件之間的溝通傳遞 | 重新認識 Vue.js | Kuro Hsu](https://book.vue.tw/CH2/2-2-communications.html)

- render props vs scoped-slot

[Modern component reusability: Render props in React & scoped slots in Vue - LogRocket Blog](https://blog.logrocket.com/modern-component-reusability-render-props-in-react-scoped-slots-in-vue-ff3c5b2dc899/)

# 錄影

[](https://drive.google.com/file/d/1dDgh1QD11TGOqcb5HiyrXVZGmATAcUu-/view)

[https://www.notion.so/react-vs-vue-4c09a5938b6547a7b391fbe94d9699ef?pvs=4](https://www.notion.so/react-vs-vue-4c09a5938b6547a7b391fbe94d9699ef)

[https://www.notion.so/react-vs-vue-4c09a5938b6547a7b391fbe94d9699ef?pvs=4](https://www.notion.so/react-vs-vue-4c09a5938b6547a7b391fbe94d9699ef)

[https://www.notion.so/react-vs-vue-4c09a5938b6547a7b391fbe94d9699ef?pvs=4](https://www.notion.so/react-vs-vue-4c09a5938b6547a7b391fbe94d9699ef)

[https://glimmer-point-bfc.notion.site/react-vs-vue-4c09a5938b6547a7b391fbe94d9699ef](https://www.notion.so/react-vs-vue-4c09a5938b6547a7b391fbe94d9699ef)
