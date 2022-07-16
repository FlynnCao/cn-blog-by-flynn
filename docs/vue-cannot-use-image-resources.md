# vue的图片资源引入不显示问题 

## 问题

使用Vue框架封装组件很多适合需要引入图片这样的静态资源；为了提高复用性， 封装一个自定义vue组件，暴露 title 和 fileName 两个属性

 
```js
// src/views/skills/components/Skill.vue
<template>
	<div>
		<img/>
		<p>{{ title }}</p>
	</div>
</template>

<script>
export default{
	name: 'Skill',
	props:{    
	 title: {
				type: String,
				required: false,
			},
			fileName: {
				type: String,
				required: false,
			}
	},
	methods:{
		// ...
	}
}
</script>

```

外部引入Skill组件，提供两个必选属性的值

```js
  <div>
		<Skill title="Java" fileName="java.png"></Skill>
	</div>
```

试图获取 `src/assets/logos` 目录下的 `java.png` 文件。

最终想要完成这样的效果：

![20220716170420](https://image-1256777099.cos.ap-beijing-fsi.myqcloud.com/2022/20220716170420.png)

> 本文示例代码使用了 `@` 符号作为根目录 `/src/` 的别名
> 

## 尝试无效：直接拼接

//skill组件的templates

```js
<div elevation="2" class="skill-card">
		<img :src="getImgPath(fileName)" alt="">
		<p>{{ title }}</p>
</div>
```

//skill组件的methods

```js
//skill组件的methods
getImgPath(fileName){
	const realPath = `@/assets/logos/${fileName}`
	return realPath
}
```

获取不到图片资源。

## 解决方式1：import导入静态资源

在外部引用组件前，通过import导入所有图片资源

Skill组件外写法：

```js
<template>
	<Skill title="Node" :file="nodei"></Skill>
</template>

<script>
import nodeImg from '@/assets/logos/node.png';
export default {
	name: 'Skills',
	components: {
		Skill,
	},
	data() {
		return {
			nodei: nodeImg
		}
	},
}
</script>

```

这时 `nodeImg` 可以当做一个文件来使用，因此img标签的 `src` 可以直接绑定之

Skill组件内这么写：

```js
<template>
<v-card elevation="2" class="skill-card">
<img :src="file" alt="">
<p>{{ title }}</p>
</v-card>
</template>

<script>
export default {
	name: 'SkillBlock',
	props: {
		title: {
			type: String,
			required: false,
		},
		file: {
			type: Object,
			required: false
		}
	},
</script>
```

最终效果：

![20220716170704](https://image-1256777099.cos.ap-beijing-fsi.myqcloud.com/2022/20220716170704.png)

## 解决方式2：require方法（commonjs规范)

以下方法是无效的：

```html
//templates

<div elevation="2" class="skill-card">
		<img :src="require(getImgPath(fileName))" alt="">
		<p>{{ title }}</p>
</div>

//methods
<script>
getImgPath(fileName){
	const realPath = `@/assets/logos/${fileName}`
	return realPath
}
</script>
```

尽管能打印出正确的路径 `@/assets/logos/java.png` 但文件系统未引用到正确的静态资源；

查阅资料后得知，***require*通常不适用于ES6的字符串文字(*模板字符串*)**

**💫 正确的解决方法：**

直接拼接字符串来向require方法中输入地址

修改 getImgPath方法为：

```js
  getImgPath(fileName){
      let url
			try {
				url = require('@/assets/logos/' + fileName)
			} catch (error) {
				url = require('@/assets/logos/default.png') //处理一下找不到图片资源的情况
			}
			return url
}    

```

完成目标。

💙 如果不考虑加载失败的情况，也可以略去try…catch或者判空的步骤（getImgPath这个函数也不需要了）。以下搭配了UI框架，以[Vuetify](https://vuetifyjs.com/)为例。

Skill组件内的template域简写为：

```js
 <v-card elevation="2" class="skill-card">
		<v-img :src="require('@/assets/logos/' + fileName)" alt="" width="100" height="100" />
		<p>{{ title }}</p>
	</v-card>
```

正常加载出来的效果：

![20220716170651](https://image-1256777099.cos.ap-beijing-fsi.myqcloud.com/2022/20220716170651.png)

## 补充

可以用这个网站去掉png图片背景烦人的网格（alpha元素），你也不想每次都用photoshop的对吧？

[Remove Background from Image - remove.bg](https://www.remove.bg/)