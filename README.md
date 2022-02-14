# notes-vue

《深入浅出Vue.js》

一。vuejs简介

渐进式框架（progressive framework):
框架分层，视图 -> 组件机制 -> 路由机制 -> 状态管理 -> 构建工具
既可以用核心功能也可以添加其他分层功能

vuejs 2.0:
引入虚拟dom，jsx，typescript，ssr

二。变化监测

Object.defineProperty 设置拦截器收集依赖和通知依赖
不能追踪key的增加和删除，只能追踪数据更改

* data通过observer转换getter/setter来追踪变化
* 数据变化时，触发setter，向dep中watcher依赖发送通知
* 外界通过watcher读取数据时，触发getter将watcher添加依赖中
* Watcher接收通知传递到外界


![Screen Shot 2021-11-16 at 3 09 05 PM](https://user-images.githubusercontent.com/38830527/153814500-d9310e62-ce43-403b-ab69-5d18d6daf4eb.png)

追踪变化代码实现：



三。 数据变化监测

利用array.prototype 设置拦截器，拦截数组方法（push, pop, splice…）
 object.__proto__ -> arrayMethods (拦截器:arrayMethods)
 arrayMethods.__proto__ -> Array.prototype

vuejs 兼容 es5：
'__proto__' in {} // 检测是否含有__proto__ 的key
如果有：用__proto__指向Array.prototype来继承，value.__proto__ = arrayMethods
如果没有：复制所有Array方法到相应对象的拦截器（arrayMethods）上

四。 变化监测api

- 所有以$开头的方法都是在Vue的原型上设置

1. $watch:	vm.$watch(expOrFn, cb, [options])

[options]: immediate:true
Passing in immediate: true in the option will trigger the callback immediately with the current value of the expression
触发回调当watcher初始化（不需要等watcher监测变量变化）

[options]: deep:true
递归收集对象依赖_traverse()

2. $set:	vm.$set(target, key, value)

如果target是array：通过splice的数组拦截器添加响应依赖
如果target是object：通过defineReactive对新增prop增加响应依赖

3. $delete: 	vm.$delete(target, key)

如果target是array：通过splice的数组拦截器删除响应依赖
如果target是object：通过delete target[key] & target.__ob__.dep.notify()删除响应依赖并通知


五。虚拟DOM

- 由jquery的命令是操作DOM到通过描述状态和DOM之间映射关系的响应式操作
- Vue中通过模版来描述状态与试图之间的映射关系，先将模版变异成渲染函数，执行渲染函数生成虚拟节点，最后使用虚拟节点更新视图
- Angular 脏检查， react 虚拟dom， vue 动态粒度更新渲染（以组件为最小粒度重新渲染组件中变化的节点）的虚拟dom

六。VNode

- 虚拟DOM实例VNode
![81 export default class VMod](https://user-images.githubusercontent.com/38830527/153814574-f01307b7-804a-449a-8a97-874b6812c664.png)

1. 注释节点：
![text 運释节点”，](https://user-images.githubusercontent.com/38830527/153814582-e29c2ee2-6f32-4a42-8f0a-419917b81d68.png)

2. 文本节点
![text“Hello Berwin](https://user-images.githubusercontent.com/38830527/153814585-d8c288dc-91d5-4260-ad6b-4ecf137f78cb.png)

3. 克隆节点
作用是优化静态节点和插槽节点：静态节点不需要重新渲染，所以使用克隆节点进行渲染不需要重新执行渲染函数
4. 元素节点
![pspanHel1oxspanspanBerwinspanp](https://user-images.githubusercontent.com/38830527/153814596-f5ca7ab2-6581-4544-b74a-123903f47771.png)

5. 组件节点
![childychild](https://user-images.githubusercontent.com/38830527/153814606-63d4dc75-2902-48e9-8d97-13df33bdea30.png)

6. 函数式组件
![functionalcontext f )，](https://user-images.githubusercontent.com/38830527/153814619-f7c1a94e-46ac-40d8-8255-98ec39d47aa4.png)

七。 patch

- 利用patching算法对比新旧VNode不同，渲染虚拟DOM
- 通过创建新节点，删除节点，修改节点更新视图

1. 新增节点
对比创建新节点，替换旧节点，删除旧节点
2. 删除节点
3. 更新节点
状态变化， 生成新节点， 对比节点， 发现为同一节点更新节点VNode信息
![图 7-5 patch 运行流餐](https://user-images.githubusercontent.com/38830527/153814631-77822e29-2e41-43f1-87e3-5d80c02eb2de.png)


7.2 创建节点

![创建文本市点](https://user-images.githubusercontent.com/38830527/153814644-7e878d3b-8db0-4ac5-a380-2f0bba2abb00.png)


7.3删除节点

删除VNode方法：遍历需要删除的startIdx 到 endIdx然后执行删除操作
删除DOM节点方法：removeChild封装到nodeOps，直接执行删除子节点操作（封装作用是让渲染机制和DOM节藕进而可以跨平台开发）

7.4更新节点
![P 7-10 WENT WARSUA ELALTE](https://user-images.githubusercontent.com/38830527/153814650-4af22951-3d27-4923-b77f-66f3a246c154.png)

7.5 更新子节点

更新子节点4种操作：更新节点，新增节点，删除节点，移动节点

Patching 算法：
- 不设key，newCh和oldCh只会进行头尾两端的相互比较。
1. oldStart == newStart？
2. oldEnd == newEnd?
3. oldStart == newEnd?
4. oldEnd == newStart?
5. 双指针后移
- 通过双指针只操作中间未处理的节点 oldStartIdx, oldEndIdx, newStartIdx, newEndIdx
while(oldStartIdx <= oldEndIdx && newStartIdx <= newEndIdx) 有一个条件满足退出循环其余为比较的node为新增或需要删除的node

![newCh](https://user-images.githubusercontent.com/38830527/153814665-e0922a81-e484-49fa-83bc-bae15b82f929.png)


八。模版编译

模版 -》 渲染函数 -〉vnode -》 dom

步骤：
1. 解析器： 生成AST，包括html解析器，文本解析器，过滤器解析器
2. 优化器： 节约js运算，遍历AST标记静态节点，patching可以跳过，重新渲染复用克隆节点不生成新子节点
3. 代码生成器： 由AST生成渲染函数字符串 new Function( code_string )

九。解析器

解析器作用是生成AST
9.1 解析器内部原理：

Html解析器在解析过程中会不断触发钩子函数：start(tag, attires,…), end(), chars(), comment()
在钩子函数内获取构成AST节点信息
构建AST层级关系是由stack实现的，用栈来记录层级深度，当遇到关闭标签比如</span>时弹出<span>节点

9.3 html 解析器转字符串到钩子函数内参数 start(tag, attires)

- 解析内容分为几种类型：开始标签，结束标签，html注释，条件注视，文本
- 利用正则表达式匹配并截取start tag, attrs部分字符串
![const ncname  a-ZA-Z_J 11wl-V1 、](https://user-images.githubusercontent.com/38830527/153814725-212b4250-df0f-4386-a9c5-0153294c14ae.png)
![const attribute = ^ls( ^1s'V= +)(s(-)s(( ^](https://user-images.githubusercontent.com/38830527/153814734-967227a4-5fd7-4405-a9f4-320cceba911d.png)

- 纯文本元素处理区别（script, style, textarea）：当判断父级为纯文本元素tag，直接匹配tag之间的文本内容然后触发钩子函数chars

9.4 文本解析器
- 解析纯文本中变量{{variable}}，方法是匹配双大括号替换成函数_s(x)最后把返回值join拼接
![const tagRE - M(M((  IMn)+)1)1Ng](https://user-images.githubusercontent.com/38830527/153814743-d5475a18-c0db-4277-a473-e25832299c7e.png)


十。 优化器

优化器的作用是在AST中找出静态子树并打上标记，这样做有两个好处：
- 每次重新渲染时，不需要为静态子树创建新节点；
- 在虚拟DOM中打补丁的过程可以跳过。

优化器的内部实现其实主要分为两个步骤：
（1)在AST中找出所有静态节点并打上标记；
（2)在AST中找出所有静态根节点并打上标记。

通过递归的方式从上向下标记静态节点时，如果一个节点被标记为静态节点，但它的子节点却被标记为动态节点，就说明该节点不是静态节点，可以将它改为动态节点。静态节点的特征是它的子节点必须是静态节点。

标记完静态节点之后需要标记静态根节点，其标记方式也是使用递归的方式从上向下寻找，在寻找的过程中遇到的第一个静态节点就为静态根节点，同时不再向下继续查找。但有两种情况比较特殊：一种是如果一个静态根节点的子节点只有一个文本节点，那么不会将它标记成静态根节点，即便它也属于静态根节点；另一种是如果找到的静态根节点是一个没有子节点的静态节点，那么也不会将它标记为静态根节点。因为这两种情况下，优化成本大于收益。

十一。代码生成器

- 将AST转换成渲染函数的代码字符串
模版(<div id=“el”>Hello {{name}}</div>) -> 解释器生成AST -> 优化器标记静态节点 -> 代码生成器生成渲染函数（
![wlth(this)(return_c(dly,(attrs(ldel)),L_v(Hello +_s(nane)))))](https://user-images.githubusercontent.com/38830527/153814783-6bea18bf-8aa0-4095-b11f-f2dcd1f831bc.png)
）

11.1 AST生成代码字符串
- 读取AST节点数据（标签名，属性，子节点）放入对应类型VNode创建函数中生成VNode
![_c(tagnamev,datav」 children）](https://user-images.githubusercontent.com/38830527/153814800-60df252b-c294-4f54-af0d-a6c0ae559b31.png)

![treatelextm](https://user-images.githubusercontent.com/38830527/153814806-984210ba-3c1c-4e34-a2a2-c0873a73f497.png)


十二。 架构设计和项目结构

运行时+编译器（30%体积） = 完整版
![SCripts](https://user-images.githubusercontent.com/38830527/153814831-284a1374-29d3-4cfb-8e7a-5f1dd7a08d99.png)
![12 1件 不同的 We k构建版本的区别](https://user-images.githubusercontent.com/38830527/153814842-a7b9c5d2-e648-4939-aa90-d70509b40d06.png)

12.2 架构设计

![平台](https://user-images.githubusercontent.com/38830527/153814860-015a3c09-0b6f-4028-8147-ed1661221787.png)

十三。 实例方法和全局API

Vue对象构造函数先后执行函数，向原型挂载方法：

* initMixin(Vue)
* stateMixin(Vue) ：数据相关实例方法（$set, $delete, $watch）
* eventsMixin(Vue) ：事件相关实例方法（$on, $once, $off）
* lifecycleMixin(Vue)：生命周期相关实例方法（$mount, $forceUpdate, $nextTick, $destroy）
* renderMixin(Vue)

13.2 eventsMixin

Vm.$on：实现通过存储维护字典vm._events[event] = fn

Vm.$off([event, callback]): 1. 没有事件移除所有监听器；2.只有事件移除事件监听器；3.都有移除这个回调的监听器
实现：1. vm._events = Object.create(null);置空所有事件3. vm._events[event] = null；置空记录时间字典

Vm.$once：只执行一次，触发后移除监听器
实现：给回调函数添加拦截器，在拦截器中先移除监听后执行回调 vm.$on(event, intercepter)
（$off移除会把拦截器函数和回调函数，结果总会是不相同，所以要把原函数fn保存到拦截器属性中，$off是在对比cb === fn || cb.fn === fn）

vm.$emit：触发事件
实现：找到vm._events[event] 所有回调并执行cbs[I].apply(vm, args)

13.3 lifecycleMixin(Vue) 生命周期相关

$forceUpdate：vue实例重新渲染，仅影响本身和插入的插槽内容（当前模版内内容），不影响子组件
实现：手动触发vm._watcher.update()

$destroy：摧毁实例，清楚连接，解绑指令和监听器，然后触发beforeDestroy和destroyed钩子（手动调用等同于v-if）
实现：
1. 触发钩子beforeDestoryed
2. 清楚与父组件连接：在父组件的$children中找到当前组件并清除；
3. 销毁当前实例watcher：vm._watcher.teardown()；
4. 销毁用户定义的vm.$watch：清楚当前实例维护的vm._watchers[I].teardown()（每当创建watcher实例都会把watcher添加到vm._watchers中vm._watchers.push(this)）；
5.  解绑模版指令：vm.__patch__(vm._vnode, null)；
6.  触发钩子destroyed；
7. 移除事件监听：vm.$off()

$nextTick：下次DOM更新周期（下次微任务执行时）之后执行。调用公共API nextTick
注意：修改响应数据也是微任务，因此先修改数据在插入nextTick微任务才可以在回调中获取最新的DOM数据

$mount：
![包建升挂教到#app （会昝换#app)](https://user-images.githubusercontent.com/38830527/153814882-83d74513-a73f-48d9-83d7-15c546735c2e.png)
模版选项优先级：render() > el > template

完整版：
- 使用document.query获取DOM节点，之后可以吧vnode挂载到上面
- 如果没有渲染函数renderer则获取模版字符串template
- 如果用户没有通过 template 选项设置模板，那么会从 el选项中获取 HTML 字行串当作模板(getOuterHtml(el))。如果用户提供了 template 选项则会判断template提供的是#id或者是模版(template.innerHTML)
- 最后通过compileToFunctions把模版编译成渲染函数
- 把renderer赋值给this.$options

运行时版本：
- 直接获取el节点（query(el)）然后通过mountComponent挂载到DOM
- mountComponent会判断实例是否有渲染函数，如果没有则
- 会为把注释节点挂载到el上
- 挂载是持续性的（持续更新变化vnode），渲染是一次性的
![vm  watcher = new Watcher(vm, （） =＞，](https://user-images.githubusercontent.com/38830527/153814898-86b88512-a35d-47ac-9335-fa524b9f24fe.png)

13.4 全局API实现原理

13.4.1 Vue.extend (options)

1. 尝试读取缓存中的组件（sub）
2. 判断extendOptions.name是否符合规则
3. 初始化生成组件sub (this._init(options))
4. 将父类的原型继承到子类（Sub.prototype = Object.create(Super.prototype)；Sub.prototype.constructor = Sub）
5. 继承options（mergeOptions(Super.options,extendOptions)）
6. Sub中添加[‘super’]属性（Sub[‘super’] = Super）
7. 初始化props（initProps(Sub)）
8. 初始化computed（initComputed(Sub)）
9. 复制方法到子类：extend, mixin, use, component, directive, filter
10. 子类增加属性Sub.superOptions, Sub.extendOptions, Sub.sealedOptions

13.4.2 Vue.nextTick( [callback,context] )

同13.3 vm.$nextTick

13.4.3 Vue.set(target, key, value)

同vm.$set

13.4.4 Vue.delete(target, key)

同vm.$delete

13.4.5 Vue.directive(id, [definition{Function | Object}])

- this.options.directives存储组件指令
- 如果没有参数definition则为getter获取指令(Vue.directive(‘my-directive’))
- 如果definition为对象，则说明是用户自定义指令，如果是函数则为原生指令(v-show, v-model)
- 如果definition为函数，则默认监听bind和update两个事件，结果是把definition函数分别复制给对象参数的bind和update

13.4.6 Vue.filter(id, [definition{Function | Object}])

- this.options.filters存储组件过滤器

13.4.7 Vue.component(id, [definition])

- this.options.components存储组件子组件
![注册鉏件，传入一个扩限过的构造器](https://user-images.githubusercontent.com/38830527/153814912-b42a9474-12f1-4749-82ea-a9b1eeebb53c.png)


13.4.8 Vue.use(plugin)

- 安装plugin，判断plugin和plugin.install哪个是函数则运行插件安装

13.4.9 Vue.mixin(mixin)

- 把用户传入对象混入到Vue自身的options属性
- mergeOptions(this.options, mixin)

13.4.10 Vue.compile(template)

- 把模版编译成渲染函数
- 只存在完整版中
- Vue.compile = compileFunctions

13.4.11 Vue.version

- 获取Vue版本信息
- Vue.version属性是读取package.json 中的version版本号码

十四。生命周期

初始化阶段，模版编译阶段，挂载阶段，卸载阶段

1. 初始化阶段：created之前
Vue实例上初始化属性，事件，响应式数据(props,methods,data,computed,watch,provide,inject)

2. 模版编译阶段：beforeMount之前
- 该阶段只存在完整版
- 使用vue-loader或者vueify把.vue文件在构建时已经与编译成js(渲染函数)

3. 挂载阶段：mounted之前

- 把模版渲染到指定的DOM元素上
- 开启watcher追踪变化
- 更新渲染时触发beforeUpdate钩子，之后触发updated钩子

4. 卸载阶段

- 从父组件中删除
- 取消数据追踪和事件

14.2 源码了解生命周期

1.初始化阶段：new Vue()

- 生命周期的初始化流程通过执行this._init(options)
- 实现：先合并options（该组件的，父组件的和用户传入的options），然后初始化生命周期，事件，renderer，injections，state（props, methods, data…），provide
![initLifecycle(vm）](https://user-images.githubusercontent.com/38830527/153814932-984adc14-7c01-4752-8334-55d35b62c565.png)

2.callhook生命周期钩子函数实现

- 可以通过vm.$options[hook]获取钩子函数
- 因为初始化生命周期阶段会先合并options到$options，并继承父组件$options，所以这里vm.$options[hook]是一个数组
- 实现是通过遍历const handlers = vm.$options[hook] 的handlers（handlers[I].call(vm)）

14.4 初始化实例属性：Vue.initLifecycle(vm)

- 找到第一个非抽象父组件作为该组件的父组件($options.abstract)，并把该组件放到父组件的$children里
- $root为如果该父组件没有父组件，则为根节点组件
- 然后依次初始化该组件的$children=[], $refs={}, _watcher=null, _isDestroyed=false

14.5 初始化事件：Vue.initEvents(vm)

- 用于父组件v-on监听子组件的$emit()事件
- 把父组件注册的监听放到子组件的vm.$options._parentListeners中
- 通过normalizeEvent(name)把事件修饰符（.once, .capture）对应的符号解析出来（因为在模版编译阶段解析标签属性是会把这些修饰符改成对应的符号放在事件名前面比如~表示once）

14.6 初始化inject：Vue.initInjections(vm)

- Inject可以接受数组或者对象{[name]:{from:’bar’, default: ‘foo’}}
- inject在data/props（initState）之前初始化，provide在data/props之后初始化

inject内部原理：

- 从下至上搜索inject在祖先组件中内容，并把返回结果放到当前组件中
- 把返回结果放入defineReactive注册到当前实例上
- 在祖先组件中的_provided[key]中寻找inject注册的键名，并循环过程直至找到（while(source) { source=source.$parent }）

14.7 初始化状态：Vue.initState(vm)

- 初始化实例状态（props，methods，data…）
- 创建属性_watchers收集当前组件的watcher，把vm.$watch和watch()注册的watcher都放到里面
- 依次初始化initProps(), initMethods(), initData(), initComputed(), initWatch()

14.7.1.初始化props：initProps(vm, propsOptions)
- 执行渲染函数阶段当某个节点是组件节点，那么在虚拟DOM渲染过程中会将子组件实例化，并把标签上的属性作为props参数传递给子组件

1. 规格化props：数组形式的props会转化为对象格式(normalizeProps(options, vm))
- 实现数组类型props：依次读取数组中的String类型的值（props: [‘userName’]），camelize(key) 驼峰化键名，然后默认props的类型为null（{userName: {type: null}}）
- 实现对象类型props：（支持对象声明：{propA: Number}, {propB: [String, Number]}, {propC: {type: String, required: true}}）使用for…in 循环该对象，camelize(key), 如果key对应值为对象则直接赋值，如果对应值不是对象则默认{type: propsValue}

2. 初始化props

- 保存父组件传递的props（const propsData = vm.$options.propsData）
- 保存组件设置的props到vm._props私有属性
- 如果组件为根组件，把属性转换成响应式数据（toggleObserving(false)）
- 遍历propsOptions（子组件用户设置的props）把validateProps函数返回值变为响应式数据(defineReactive)放入vm._props
- validateProps：只为类型为boolean的props转换传入值（父组件没有传props，则为false）（父组件传key值，则为true）（父组件传key和val值相同字符串，有kebab命名转换后，则为true）
- validateProps实现：处理布尔类型props -> 检查默认值default并转化为响应式数据 -> 验证props（assertProp）输出警告（props: {someProp: true} 可以跳过检查）

14.7.2 初始化methods：initMethods(vm, methods)

1. 检验method是否合法：
- 只有key，发出警告
- methods中key已经存在props中，发出警告
- key已经存在vm中并且是保留命名方法（以$或_开头），发出警告
2. method挂载到vm：
- bind() 函数绑定this到vm，便可以通过vm.myMethod访问到方法

14.7.3 初始化data：（initData(vm)）

1. 获取data对象：
- 如果data是function，通过getData(data, vm)获取对象
- Data对象保存到vm._data私有字段
2. 检验data：
- Data中key是否已经存在methods中
- data中key是否已经存在props中
3. 代理到_data属性
4. 响应式data
- 执行observe(data, true /* asRootData */)

14.7.4 初始化computed：initComputed(vm, computed)


- 简单来说computed是定义在vm上的一个特殊getter方法
- 计算属性结果会被缓存(只有在非ssr环境才会缓存shouldCache)，通过结合watcher的dirty属性来分辨是否需要重新计算computed返回值
![3 14-4 HOTEL I O AGL](https://user-images.githubusercontent.com/38830527/153814962-42f30a68-f5a9-491c-9ea4-bf40202630eb.png)
![RI 14-5 BEHGHOTA MANO PUMSUH](https://user-images.githubusercontent.com/38830527/153814968-298a9497-04bd-4974-a849-d37fe52ab912.png)

代码实现：
1. 声明watchers对象用来保存computed属性的所有watcher，并代理到vm._computedWatchers
2. 遍历用户定义的computed对象，如果computed[key]为函数则直接使用当作getter函数，否则则要获取对象的get函数computed[key].get作为getter
3. 如果在非ssr环境中，为计算属性创建内部观察器：watchers[key] = new Watcher()
4. 如果key在vm中没有声明过，则运行defineComputed(vm, key, userDef)通过实现object.defineProperty把用户传入computed配置的key变为响应数据绑定到vm上
5. Assertion 检查是否computed的key已经呗data或者props声明过

- defineComputed方法会调用createComputedGetter，函数中调用watcher.depend()方法添加依赖(Dep)把组件的watcher依次添加到计算属性中用到所有状态的dep中实现向组件发送更新通知
- 计算属性在Vuejs 2.5.17之前计算属性中的状态发生变化单最终值没有变化组件也会走一遍渲染流程

新版本vuejs计算属性：
- 使用组件的watcher观察计算属性watcher，然后使用计算属性watcher观察computed中的数据
- 如果是用户自定义的watcher（vm.$watch），则使用自定义watcher观察计算属性watcher

14.7.5 初始化watch：initWatch

- 用户自定义watch对象{[key: String]: String | Function | Object | Array}
- 实现方式是通过vm.$watch(expOrFn, handler, options)

14.8 初始化provide：initProvide(vm)

- 实现方式：把provide函数返回值放入父组件vm._provided属性中，子孙组件可以直接访问到

十五。指令的奥秘

- 指令是vue提供带有v-前缀的特殊属性，并影响于DOM
- 通过Vue.directive全局api声明
- 在patch阶段会对比新旧节点并触发一些钩子函数，更新指令程序会监听create，update，destroy钩子函数，最终根据对比结果触发指令的钩子函数（bind, inserted, update, componentUpdated, unbind）并执行函数体内容
![校板解析](https://user-images.githubusercontent.com/38830527/153814978-cb1a36a8-b08a-48ca-b7d8-92d4434de170.png)


15.1.1 v-if

- 一些内置指令是在模版编译阶段实现的
- v-if可以编译成渲染函数并转换成三元判断

＜li v-if="has"> if </li>
＜li v-else> else </li>

（has) 
	? _c ('1i', [_v("if")])
	:_c('1i',[_v("else")])

15.1.2 v-for

 <li v-for-"(item,index) in list"> v-for {{index}}</li>

_l ( (list) ,function (item, index) {
	return_c(‘li’, [
		 _v (“v-for“＋＿s(index))
		])
	})
- _l（renderList）, _v（创建文本节点函数）

15.1.3 v-on

- v-on在模版解析阶段保存到VNode上可以通过vnode.data.on得到一个节点注册的所有事件
- 事件绑定相关事件设置在DOM patch阶段的create和update钩子，每当DOM创建和更新都会触发事件绑定逻辑
- 事件绑定相关函数updateDOMListeners（oldVnode, vnode）：对dom绑定事件对比然后调用updateListeners更新事件监听器来判断添加或解绑事件等
- 绑定和解绑事件监听是通过浏览器原生函数EventTarget.addEventListener 和 EventTarget.removeEventListener

15.2 自定义指令

- create 和 update钩子函数调用updateDirectives(oldVnode, vnode)，最终调用_update(oldVnode, vnode)函数来处理指令 

实现：
1.  调用normalizeDirectives函数同一格式化oldDirs和newDirs数组保存新旧directives
2. 遍历newDirs和oldDirs并保存到dirsWithInsert和dirsWithPostpatch数组中为之后触发
3. 遍历过程中如果是新指令触发bind钩子函数，如果不是新指令触发update钩子函数
4. 如果有dirsWithInsert，触发钩子函数inserted，再如果是新创建的虚拟节点通过mergeVNodeHook方法（把钩子函数于虚拟节点合并排序，保证先执行组件钩子函数再执行指令钩子）也触发inserted钩子
5. 如果有dirsWithPostpatch，通过mergeVNodeHook方法触发componentUpdated钩子函数
6. 如果上述条件都并满足并且不是新创建节点，对比新旧指令集合数组，在旧指令集中遍历哪些不存在新指令集中，触发unbind钩子函数

- bind: 新指令被定义初始化阶段（inserted之前）
- inserted: 新指令被插入，在被绑定元素插入到父节点之后调用
- update: 已存在指令更新初始化阶段（componentUpdated之前）
- componentUpdated: 旧指令被更新，所在组件的vnode和子组件的vnode全部更新后
- unbind: 指令与元素解绑时，不存在于新虚拟节点的指令列表中

15.3 虚拟DOM钩子函数

![activate eepAlive 州件微创理](https://user-images.githubusercontent.com/38830527/153814997-2d4bc3cc-a3c7-4279-8175-c3d108d26d46.png)

十六。过滤器的奥秘

- 可以设置全局过滤器和全局过滤器Vue.filter()
- 链式操作，用管道pipe分割

16.1 过滤器原理：

｛ {message| capitalize } }

＿s (_f ("capitalize") (message))

- _f：resolveFilter（’capitalize’），作用时从this.$options.filters[‘capitalize’]找出过滤器
- 高阶函数_f返回的函数再传入参数message
- _s：toString，把最后结果函数转换成字符串

16.2 解析过滤器

function parseFilters(exp) {
  let filters = exp.split('|')
  let expresssion = filters.shift().trim()
  let i
  if (filters) {
    for (i = 0; i < filters.length; i++) {
	// recursively wrap expression
      expresssion = wrapFilter(expresssion, filters[i].trim())
    }
  }
  return expresssion
}
function wrapFilter(exp, filter) {
  const i = filter.indexOf('(')
  if (i < 0) {
    // no second argument
    return `_f("${filter}")(${exp})`
  } else {
    const name = filter.slice(0,i)
    const args = filter.slice(i + 1)
    return `_f("${name}")(${exp},${args})`
  }
}

let r1 = parseFilters('message | capitalize')
console.log(r1); // _f("capitalize")(message)
let r2 = parseFilters('message | filterA|filterB')
console.log(r2); // _f("filterB")(_f("filterA")(message))
let r3 = parseFilters('message | filterA("arg1",arg2)')
console.log(r3); // _f("filterA")(message,"arg1",arg2))

十七。最佳实践

- Vuejs官方风格指南：https://cn.vuejs.org/v2/style-guide/

1. 渲染列表增加key，提高虚拟DOM算法更新速度，提高性能
2. v-if/v-else中添加key, 彻底替换旧元素而不是修补已存在元素，减少副作用
3. 路由切换到不同参数地址不会触发生命周期，因为vue-router会识别出路由路径使用的是相同组件，可以复用，不会重新创建新组件，所以不会触发生命周期
- 通过beforeRouteUpdate钩子函数解决，在路由改变且被复用时触发
- 通过watch监听watch:{ ‘$route’(from,to){} }中监听路由变化
- 通过在router-view上添加key，<router-view :key=“$route.fullPath”” />，当key改变vue会重新创建这个组件
4. 通用组件使用props和事件通讯而不是vuex或store，可以与业务逻辑解藕
5. v-if和v-for同时使用
- v-for比v-if具有更高优先级，且在每次渲染时需要遍历整个列表
- 当要过滤列表元素时使用，可以添加computed属性先过滤列表再放到模版中渲染，还可以解藕渲染层逻辑，增加可维护性，列表只有在数组变化时才会重新运算更高效过滤，更高效渲染
- 当需要隐藏整个列表时，可以把v-if添加到容器上
6. 组件样式设置作用域：scoped和css modules（基于class类的BEM策略）（类名会编译成._src_components_HelloWorld__buttonClose）css module可以更容易复写内部样式
<template>
  <button :class="[$style.button, $style.buttonClose]">x</button>
</template>

<style module>
.button {
  border: none;
  border-radius: 2px;
}
.buttonClose {
  background-color: red;
}
</style>
7. 避免在scoped中使用元素选择器：类选择器比元素选择器更快（元素选择器会编译成button[data-v-f3f3eg9]）
8. 避免隐性父子通讯：应该使用props和事件，不应该使用this.$parent
9. 单文件组件命名规范：
- 始终使用同一命名规范首字母大写（PascalCase）或者横线连接（kebab-case）
- 优点：对代码编辑器自动补全更友好
10. 基础组件名：以特定的前缀开头，优点：编辑器字母排序所有基础组件会列在一起，把所有基础组件引入到全局时更容易编写webpack
11. 单例组件名：（每个页面只引用一次）以The为前缀，并且没有props属性
12. 紧密耦合的组件名：子组件应以父组件名作为前缀
- TodoList.vue/TodoListItem.vue
- 优点：避免太多文件名重名，可以更快的搜索文件；扁平化文件夹结构，更好的在编辑器侧边栏寻找
13. 组件名单词顺序：高级别的（一般化描述）开头，修饰性结尾
- SearchButtonClear.vue | SearchButtonRun.vue
14. 完整单词组件名：避免不常用的缩写，编辑器有强大的自动补全功能
15. 组件名为多个单词：避免与html标签元素冲突（html元素名都是单个单词）
16. 模版中组件名首字母大写（PascalCase）：编辑器模版自动补全组件名，更能与html元素区别开
17. 自闭和组件：<MyComponent/> | <my-component></my-component>
18. Props名声明与使用：命名用驼峰式（camelCase），模版使用用横线链接（kebab-case）
- props: { greetingText: String } | <WelcomeMessage greeting-text=“hi” />
19. 模版中只包含简单表达式：模版中应只描述应该出现什么而不是怎么计算，可以使用计算属性和方法计算数据
20. 简单的计算属性：把复杂的计算属性分割为简单的计算属性
- 优点：易于测试；易于阅读；更好延展性，分割出的简单计算属性也可以用在以后的视图上
21. 统一的指令缩写：全部使用或者全部不使用
22. 组件/实例选项顺序：影响由外至内，模版相关至js相关
23. 模版中元素attr顺序：定义，列表渲染，条件渲染，渲染方式，全局感知id，唯一attr，v-model，其他，事件，覆盖元素内容

