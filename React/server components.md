# React Server Components

## RSC是什么

> React Server Components（RSC）是一项令人兴奋的新特性，它将对**页面加载性能**、**包大小**以及React 应用程序的编写方式产生巨大影响
>
> RSC 使得服务端和客户端（浏览器）可以**协同渲染**React应用程序，从而实现了部分组件在服务端或客户端两者之间的渲染。下面是React团队的一个插图，展示了最终目标:一个React树，其中橙色的组件渲染在服务端上，蓝色的组件渲染在客户端上

<img src="https://raw.githubusercontent.com/ilmangoi/imgRepo/main/img-2/76240a64508f42978808e70bda3c8d04~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp" alt="image.png" style="zoom: 50%;" />

## RSC与SSR的区别

### 联系

一般来讲，传统的SSR免不了两个过程，服务端渲染 + 客户端hydrate二次渲染：

- 服务端渲染：在服务端渲染出首屏内容（HTML 字符串）

- 客户端hydrate二次渲染：在客户端加载完首屏内容（此时页面不可交互）、以及前端应用的完整代码之后，进行一次类似于render的hydrate二次渲染过程，把交互事件绑上去（此时页面可交互）

Server Components的渲染过程与之类似：

- 服务端渲染：在服务端渲染出首屏内容（一种中间形式，同样是用来描述UI的）

- 客户端渲染：接到服务端输出的中间形式，从头render，开始流式渲染

所以，Server Components与SSR的联系至少有以下几点：

- 都在服务端执行组件渲染逻辑（所以都不支持交互）

- 都允许同一组件跨客户端、服务端运行（Shared Components）

- 都从客户端延伸到服务端寻求更多的性能突破

对于二者之间的关系，React官方有一个词表述得很到位，Server Components与SSR是互补的（complementary），SSR能把首屏渲染成HTML加速内容展示，Server Components能够帮助减少hydrate二次渲染所需加载执行的代码量（Server Components只在服务端渲染，相关代码不需要在客户端加载执行），进而加快页面的可交互时间

### 区别

- Server Components相关代码根本不会给到客户端，而传统SSR所有组件代码全都要打进客户端bundle里

- Server Components允许在组件树的任意位置直接访问后端，传统SSR只允许在顶层（页面级）获取数据

- Server Components在更新时能保留客户端交互状态（包括输入的搜索词、滚动位置、焦点、选中内容等等），因为Server Components渲染结果是一种比HTML信息更丰富的中间格式（毕竟HTML只能表达 HTML，自定义格式则不存在这个限制，比如能带上props）

- Server Components只在服务端执行，客户端并不加载这些代码，服务端给到客户端的始终只是Server Components的渲染结果，包括二次更新，以中间形式给到客户端后，客户端只把来自服务的渲染结果merge到当前已经渲染好的客户端组件上，所以能保留交互状态

## RSC的优势

### 更小的Bundle体积

通常情况下，我们在前端开发上使用很多依赖包，但实际上这些依赖包的引入会增大代码体积，增加bundle加载时间，降低用户首屏加载的体验

例如在页面上渲染`MarkDown`，我们不得不引入相应的渲染库，而且往往这种大型第三方类库是没办法进行tree-shaking

可以想象，为了某一个计算任务，我们需要将大型js第三方库传输到用户浏览器上，浏览器再进行解析执行它来创造计算任务的runtime, 最后才是计算。从用户的角度来讲：「我还没见到网页内容，你就占用了我较大带宽和CPU资源，是何居心」。然而这一切都是可以省去的，我们可以利用SSR让React在服务端先渲染，再将渲染后的html发送给用户。从这一方面看，Server Component和SSR很类似，但不同的是SSR只能适用于首页渲染，Server Component在用户交互的过程中也是服务端渲染，Server Component传输的也不是 html文本，而是json。Server Component在服务端渲染好之后会将一段类React组件json数据发送给浏览器，浏览器中的React Runtime接收到这段json数据后，将它渲染成 HTML

<img src="https://raw.githubusercontent.com/ilmangoi/imgRepo/main/img-2/118c6c28266745e2877760c8ae14ea9c.png" alt="在这里插入图片描述" style="zoom:50%;" />

举一个更加极端的例子：若用户无交互性组件，所以组件都可以在服务端渲染，那么所有UI渲染都将走【浏览器接收类react element文本格式的数据，React Runtime渲染】的形式进行渲染。那么除了一些Runtime， 无需其他JS Bundle。而Runtime的体积是不会随着项目的增大而增大的，这种常数系数级体积也可以称为“Zero-Bundle-Size”。因此官方这称为: “Zero-Bundle-Size Components”

### 更好的使用服务端能力

为了获取数据，前端通常需要请求后端接口，这是因为浏览器是没办法直接访问数据库的。但既然我们都借助服务端的能力了，那我们当然可以直接访问数据库，React在服务器上将数据渲染进组件

通过自由整合后端能力，我们可以解决:「网络往返过多」和「数据冗余」问题。甚至我们可以根据业务场景自由地决定数据存储位置，是存储在内存中、还是存储在文件中或者存储在数据库

除了数据获取，还可以在Server Component的渲染过程中将一些高性能计算任务交付给其他语言，如 C++，Rust。这不是必须的，但你可以这么做

简单粗暴一点的说：Nodejs 拥有什么样的能力，你的组件就能拥有什么能力。*

### 更好的自动化Code Split

在过去，我们可以通过React提供的lazy + Suspense进行代码分割。这种方案在某些场景(如SSR)下无法使用，社区比较成熟的方案是使用第三方类库`@loadable`。然而无论是使用哪一种，都会有以下两个问题：

1. Code Split需要用户进行手动分割，自行确认分割点
2. 与其说 Code Split，其实更偏向懒加载。也就是说，只有加载到了代码切割点，我们才会去即时加载所切割好的代码。这里还是存在一个加载等待的问题，削减了code split给性能所带来的好处

React核心团队所提出Server Component可以帮助我们解决上面的两个问题

- React Server Component将所有Client Component的导入视为潜在的分割点。也就是说，你只需要正常的按分模块思维去组织你的代码。React会自动帮你分割

```jsx
import ClientComponent1 from './ClientComponent1';
function ServerComponent() {
  return (
    <div>
      <ClientComponent1 />
    </div>
  )
}
```

- 框架侧可以介入Server Component的渲染结果，因此上层框架可以根据当前请求的上下文来预测用户的下一个动作，从而去「预加载」对应的js代码

### 避免高度抽象所带来的性能负担

React server component通过在服务器上的实时编译和渲染，将抽象层在服务器进行剥离，从而降低了抽象层在客户端运行时所带来的性能开销

举个例子，如果一个组件为了可配置性，被多个wrapper包了很多层。但事实上这些代码最终只是渲染为一个`<div>`。如果把这个组件改造为server component的话，那么我们只需要往客户端返回一个`<div>`字符串即可。下面例子，我们通过把这个组件改造为server component，那么我们大大降低网络传输的资源大小和客户端运行时的性能开销

```jsx
// Note.server.js
// ...imports...
function Note({id}) {
  const note = db.notes.get(id);
  return <NoteWithMarkdown note={note} />;
}

// NoteWithMarkdown.server.js
// ...imports...
function NoteWithMarkdown({note}) {
  const html = sanitizeHtml(marked(note.text));
  return <div ... />;
}

// client sees:
<div>
  <!-- markdown output here -->
</div>
```

### 与SSR互补

单从SSR的角度来看，Server Components是组件化框架从组件系统层面解决了SSR应用框架解决不了的问题，比如：

- 加快hydrate二次渲染（减少客户端所需加载执行的代码量）

- 流式渲染支持（不同于SSR流式输出，流式渲染一定是需要组件化框架本身配合服务端的）

- 允许局部刷新，保留交互状态（传统SSR只能用作首屏）

## 高层次的全貌

> React Server Component的目的是实现更好的任务分工，让服务器先处理它擅长的事情，然后将剩余任务交给浏览器完成。这样一来，服务器需要传输的内容就减少了，不再需要传输整袋面粉和一个烤箱，只需传输12个小蛋糕即可提高效率
>
> 在考虑页面中React树时，有些组件在服务器上渲染，而另一些则在客户端渲染。为了实现更高级别的策略，在“Render” server component时可以采用简化方法将React组件转换为原生HTML元素（如`div`和`p`）。但每当遇到要在浏览器中渲染的“Client”组件时，则只输出占位符，并指示使用正确的client component和props来填充该空缺。随后浏览器接收输出并使用client component填充这些占位符

React团队根据编写组件的文件扩展名进行定义：如果文件以“.server.jsx”结尾，则包含服务器组件；如果以“.client.jsx”结尾，则包含客户端组件。如果两者都没有，则该组件既可以作为服务器组件使用，也可以作为客户端组件使用。这个定义很实用——对人和打包工具来说都很容易区分

特别是对于打包工具来说，它们现在可以通过检查客户端组件的文件名来加以区别处理。正如我们即将看到的那样，在使RSC（Server Component）工作方面，打包工具发挥着重要作用。因为server component运行在服务器上，客户端组件运行在客户端上，所以每个组件可以做的事情都有很多限制。但是要记住的最重要的一点是客户端组件不能导入server component!这是因为server component不能在浏览器中运行，并且可能存在无法在浏览器中正常工作的代码;如果客户端组件依赖于server component，那么我们最终会将这些非法依赖项打包到浏览器中:

```jsx
// ClientComponent.client.jsx
// NOT OK:
import ServerComponent from './ServerComponent.server'
export default function ClientComponent() {
  return (
    <div>
      <ServerComponent />
    </div>
  )
}
```

如果客户端组件无法导入服务器组件，就无法实例化服务器组件。那么我们该如何在React树中将这样的服务器和客户端组件交织在一起呢？又该如何将橙色点的服务器组件置于蓝点的客户端组件之下？

<img src="https://raw.githubusercontent.com/ilmangoi/imgRepo/main/img-2/04ac9ba4e4a04f8884b1366356cb438f~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp" alt="A React tree with server components (orange) and client components (blue)" style="zoom:50%;" />

虽然客户端组件无法直接导入和渲染服务器端组件，但可以使用**组合**的方式。也就是说，客户端组件仍然可以接收不透明的 React 节点，并且这些节点可能正好由服务器端组件渲染。例如：

```jsx
// ClientComponent.client.jsx
export default function ClientComponent({ children }) {
  return (
    <div>
      <h1>Hello from client land</h1>
      {children}
    </div>
  )
}

// ServerComponent.server.jsx
export default function ServerComponent() {
  return <span>Hello from server land</span>
}

// OuterServerComponent.server.jsx
// OuterServerComponent can instantiate both client and server
// components, and we are passing in a <ServerComponent/> as
// the children prop to the ClientComponent.
import ClientComponent from './ClientComponent.client'
import ServerComponent from './ServerComponent.server'
export default function OuterServerComponent() {
  return (
    <ClientComponent>
      <ServerComponent />
    </ClientComponent>
  )
}
```

## RSC渲染原理

### 渲染入口

> 浏览器请求到HTML后，请求入口文件-main.js，里面包含了**React Runtime**与**Client Root**

```jsx
import { createFromFetch } from 'react-server-dom-webpack'
function ClientRootComponent() {
  // fetch() from our RSC API endpoint.  react-server-dom-webpack
  // can then take the fetch result and reconstruct the React
  // element tree
  const response = createFromFetch(fetch('/rsc?...'))
  return <Suspense fallback={null}>{response.readRoot() /* Returns a React element! */}</Suspense>
}
```

Client root就是react在浏览器中将RSC流转换成实际React元素的关键，它在其中引用了**react-server-dom-webpack**包，使用`react-server-dom-webpack`从API端点读取RSC响应。使用response.readRoot()方法可以返回一个React元素，它会随着响应流的处理而更新。在任何流被读取之前，该方法会立即抛出一个promise，因为还没有准备好任何内容。当处理第一个J0时，它创建了相应的React元素树并解决了抛出的promise。React恢复渲染过程，但遇到尚未准备好的@3引用时，另一个promise被抛出。一旦读取J3后，该promise就会得到解决，并且React再次恢复渲染直至完成。因此，在我们流式传输RSC响应时，我们将继续通过Suspense边界定义的块更新和渲染已有的元素树直至结束

由于渲染是在服务器上完成的，因此，在进行路由切换时，需要向服务器发出另一个API调用以获取新内容的RSC Wire Format。一旦浏览器接收到新内容，它就可以构建一个新的 React元素树，并执行与先前元素树的常规**协调差异**来确定DOM所需的**最小更新**。同时保留状态和事件处理程序在客户端组件中

对于客户端组件而言，这种更新与完全在浏览器中进行没有什么区别。目前必须重新渲染整个React树，但将来可能会针对子树执行此操作

### 请求服务端组件

Client Root代码执行后，浏览器会向服务端发送一个带有data数据的请求，服务端接收到请求，则进行服务端渲染

服务端将从Server Component Root开始渲染，一颗混合组件树将在服务端渲染成一个巨大的VNode

<img src="https://img-blog.csdnimg.cn/da37f3f781e84f7090856047b598cf06.png" alt="在这里插入图片描述" style="zoom:50%;" />

如图，这一颗混合组件树会被渲染成这样一个对象，它带有React组件所有必要的信息

```js
module.exports = {
  tag: 'Server Root',
  props: {...},
  children: [
    { 
      tag: "Client Component1", 
      props: {...},
      children: [] 
    },
    { 
      tag: "Server Component1", 
      props: {...},
    	children: [
    		{ 
    			tag: "Server Component2", 
    			props: {...},
  				children: [] 
				},
  			{ 
  				tag: "Server Component3", 
  				props: {...},
          children: [] 
        }
  		]
		}
	]
}
```

不仅仅是这样一个对象, 由于Client Comopnent需要 Hydration, React会将这部分必须要的信息也返回回去。React最终会返回一个可解析的Json序列Map

```
M1:{"id":"./src/BlogMenu.client.js","chunks":["client0"],"name":"xxx"}
J0:["$","main", null, ["]]
```

- **M:** 代表Client Comopnent所需的Chunk信息
- **J:** 代表Server Compnent渲染出的类react element格式的字符串

### React Runtime渲染

组件数据返回给浏览器后，React Runtime开始工作，将返回的VNode渲染出真正的HTML。与此同时发出请求，请求Client Component所需的JS Bundle。当浏览器请求到Js Bundle后，React就可以进行选择性Hydration（Selective Hydration）。需要注意的是React团队传输组件数据选择了流式传输，这也意味着React Runtime无需等待所有数据获取完后才开始处理数据

<img src="https://img-blog.csdnimg.cn/0022481aa3304f43b5a3559b8f935700.png" alt="在这里插入图片描述" style="zoom:50%;" />

### 启动流程

1. 浏览器加载React Runtime, Client Root等js代码
2. 执行Client Root代码，向服务端发出请求
3. 服务端接收到请求，开始渲染组件树
4. 服务端将渲染好的组件树以字符串的信息返回给浏览器
5. React Runtime开始渲染组件且向服务端请求Client Component Js Bundle进行选择性Hydration（注水)

<img src="https://img-blog.csdnimg.cn/18c0cf5a6b5845439b91315c81182c26.png" alt="在这里插入图片描述" style="zoom:50%;" />

## RSC渲染流程

### Server接收到render请求

由于Server需要进行预渲染，因此RSC的渲染流程始终从服务器开始。它响应一些API调用来渲染React组件。应用的根组件总是一个server component，可以渲染其他server或client组件。服务器根据传递到请求中的信息确定使用哪个server component和什么props。通常以特定URL的页面请求形式出现

### Server将root component序列化为JSON

我们的最终目标是将初始根服务器组件渲染为基本HTML标记和客户端组件“占位符”的树。然后我们可以对该树进行序列化并发送给浏览器。浏览器会反序列化该树，并使用真正的客户端组件填充占位符，从而渲染最终结果

因此以前面提到的例子为例，如果我们想要渲染这个组件，则仅使用JSON.stringify()无法获得一个序列化的元素树。这是因为React元素不像HTML标签那样简单明了，它是一个封装好的对象：

```js
> React.createElement("div", { title: "oh my" })
{
  $$typeof: Symbol(react.element),
  type: "div",
  props: { title: "oh my" },
  ...
}

// React element for <MyComponent>oh my</MyComponent>
> function MyComponent({children}) {
    return <div>{children}</div>;
  }
> React.createElement(MyComponent, { children: "oh my" });
{
  $$typeof: Symbol(react.element),
  type: MyComponent  // reference to the MyComponent function
  props: { children: "oh my" },
  ...
}
```

当你使用一个组件元素而不是基本的HTML标签元素时，type字段引用了一个组件函数。这些函数无法直接进行JSON序列化。为了正确地对所有内容进行JSON-Stringify，React 会向JSON.stringify()传递一个特殊的替换函数以正确处理这些组件函数引用。具体来说在序列化React元素时：

- 如果它被用于基本的**HTML标签**（type字段是字符串，如 "div"），那么它已经可以被序列化，无需做其他事情
- 如果它被用于**服务器端组件**，则调用其props中存储的服务器端组件函数（存储在type字段中），并将结果序列化。这有效地“渲染”了服务器端组件；目标是将所有服务器端组件转换成基本的HTML标签
- 如果它被用于**客户端组件**，则实际上已经可以被序列化。type字段实际上指向一个**模块引用对象**，而不是一个组件函数

#### 模块引用对象

RSC引入了一种新的可能值作为React Element的type字段，称为“模块引用”。这个值不同于之前的component function，而是一个可序列化的“引用”。 举例来说，client component元素可能长成这样：

```jsx
{
  $$typeof: Symbol(react.element),
  // The type field  now has a reference object,
  // instead of the actual component function
  type: {
    $$typeof: Symbol(react.module.reference),
    // ClientComponent is the default export...
    name: "default",
    // from this file!
    filename: "./src/ClientComponent.client.js"
  },
  props: { children: "oh my" },
}
```

这种变化发生在在打包环节，客户端组件函数的引用被转换为可序列化的“模块引用”对象

React团队已经发布了对webpack官方RSC支持，可以作为webpack loader或node-register使用。当服务器组件从.client.jsx文件中导入内容时，并没有真正得到该文件导出的内容，而是得到一个包含该文件名和导出名的模块引用对象。因此，在Server端构建React Tree时，客户端组件不会被包括进去。再次考虑上面的例子，当我们试图序列化组件时，最终会得到一个JSON树：

```jsx
jsx
复制代码{
  // The ClientComponent element placeholder with "module reference"
  $$typeof: Symbol(react.element),
  type: {
    $$typeof: Symbol(react.module.reference),
    name: "default",
    filename: "./src/ClientComponent.client.js"
  },
  props: {
    // children passed to ClientComponent, which was <ServerComponent />.
    children: {
      // ServerComponent gets directly rendered into html tags;
      // notice that there's no reference at all to the
      // ServerComponent - we're directly rendering the `span`.
      $$typeof: Symbol(react.element),
      type: "span",
      props: {
        children: "Hello from server land"
      }
    }
  }
}
```

#### 可序列化的React tree

在这个过程结束时，我们希望最终得到一个在服务器上看起来更像React树的结构，并将其发送到浏览器以完成页面渲染

<img src="https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5b48e21136c84d0cbe4d3e9e6a610647~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp" alt="A React tree with server components rendered to native tags, and client components replaced with placeholders" style="zoom:50%;" />

当我们将整个React树序列化为JSON时，传递给客户端的组件或基本HTML标签的所有props也必须是可序列化的。因此不能将事件处理程序作为props从服务器组件传递

```jsx
// NOT OK: server components cannot pass functions as a prop
// to its descendents, because functions are not serializable.
function SomeServerComponent() {
  return <button onClick={() => alert('OHHAI')}>Click me!</button>
}
```

在RSC过程中，当遇到客户端组件时，我们不会调用该组件，而是保存对它的引用。因此可以通过另一个客户端组件来传递事件处理程序

```jsx
function SomeServerComponent() {
  return <ClientComponent1>Hello world!</ClientComponent1>;
}

function ClientComponent1({children}) {
  // It is okay to pass a function as prop from client to
  // client components
  return <ClientComponent2 onChange={...}>{children}</ClientComponent2>;
}
```

在这个RSC JSON树中，根本没有出现Client Component2。相反，我们只能看到一个带有模块引用和Client Component1属性的元素。因此，将事件处理程序作为参数传递给ClientComponent2是完全合法的

### 浏览器重新构建React Tree

浏览器接收到Server输出的JSON后，需要重建React树以在浏览器中渲染。当遇到类型为模块引用的元素时，我们希望将其替换为真实客户端组件功能的引用

这项工作需要打包工具（bundler）的帮助。我们的bundler用服务器上的模块引用替换了客户端组件函数，现在它知道如何在浏览器中使用真正的客户端组件函数来替换那些模块引用。重构后的React树只交换了原生标签和客户端组件，看起来像这样： 

<img src="https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4ed21a5d82db4098a9c252e3af8e7c75~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp" alt="A React tree reconstructed in the browser with only native tags and client components" style="zoom:50%;" />

### RSC与Suspense

> suspense在上述所有步骤中都起着至关重要的作用。有了suspense组件的特性，就可以通过服务器流式传输RSC输出来提高性能。这样服务器组件就可以获取数据并逐步渲染它们，同时根据需要动态获取客户端组件的bundle

当React组件需要等待某些尚未准备好的东西（例如获取数据或延迟导入组件）时，suspense允许你从组件中抛出promise。这些promise被捕获在“suspense边界”处 —— 每当从渲染suspense子树抛出promise时，React会暂停渲染该子树直到promise解决后再次尝试（react18新特性：异步组件）

当我们调用服务器端组件函数以生成RSC输出时，这些函数可能会抛出promise，因为它们需要获取所需数据。遇到此类promise时，我们输出一个占位符；一旦解决了该promise，则尝试再次调用服务器端组件函数，并在成功后输出完成块。实际上，我们正在创建RSC输出流，在抛出promise期间暂停，并在其解决后流式传输其他块

在浏览器中，我们正在使用fetch()方法从上方流式传输RSC JSON输出。如果输出中遇到了服务器抛出的promise导致的suspense占位符，并且在流中还没有看到占位符内容，则此过程也可能会抛出一个promise（需要注意一些细节）。另外，如果它遇到客户端组件模块引用但尚未加载该客户端组件函数，则也可以抛出一个promise —— 在这种情况下，打包程序运行时将必须动态获取所需的块

## RSC Wire Format(数据传输格式)

> react服务器采用了一种简单的数据传输格式，在每行都包含一个带有ID标记的JSON块。以下是示例RSC输出的内容

```json
M1:{"id":"./src/ClientComponent.client.js","chunks":["client1"],"name":""}

J0:["$","@1",null,{"children":["$","span",null,{"children":"Hello from server land"}]}]
```

- M行以M开头，定义了客户端组件模块的引用。该行提供了查找客户端包中组件函数所需的信息
- J行以J开头，定义了实际的 React 元素树。其中包括由M行定义的客户端组件引用（例如 @1）

这种格式非常适合流式传输 —— 一旦客户端读取整个行，它就可以解析JSON片段并取得进展。如果服务器在渲染时遇到suspense boundaries，则会看到多个J行对应于每个块的解决

**示例：**

```jsx
// Tweets.server.js
import { fetch } from 'react-fetch' // React's Suspense-aware fetch()
import Tweet from './Tweet.client'
export default function Tweets() {
  const tweets = fetch(`/tweets`).json()
  return (
    <ul>
      {tweets.slice(0, 2).map((tweet) => (
        <li>
          <Tweet tweet={tweet} />
        </li>
      ))}
    </ul>
  )
}

// Tweet.client.js
export default function Tweet({ tweet }) {
  return <div onClick={() => alert(`Written by ${tweet.username}`)}>{tweet.body}</div>
}

// OuterServerComponent.server.js
export default function OuterServerComponent() {
  return (
    <ClientComponent>
      <ServerComponent />
      <Suspense fallback={'Loading tweets...'}>
        <Tweets />
      </Suspense>
    </ClientComponent>
  )
}
```

下面是该组件的RSC流：

```json
M1:{"id":"./src/ClientComponent.client.js","chunks":["client1"],"name":""}

S2:"react.suspense"

J0:["$","@1",null,{"children":[["$","span",null,{"children":"Hello from server land"}],["$","$2",null,{"fallback":"Loading tweets...","children":"@3"}]]}]

M4:{"id":"./src/Tweet.client.js","chunks":["client8"],"name":""}

J3:["$","ul",null,{"children":[["$","li",null,{"children":["$","@4",null,{"tweet":{...}}}]}],["$","li",null,{"children":["$","@4",null,{"tweet":{...}}}]}]]}]
```

在J0行中，现在有一个额外的子项 —— 新的`Suspense`边界，其中children指向引用@3。有趣的是在这里@3还尚未被定义！当服务器完成加载Tweets时，它会输出M4这行 —— 它定义了对Tweet.client.js组件的模块引用。以及J3 —— 它定义了另一个React元素树，应该替换@3所在位置（请注意，J3的children引用了在M4中定义的Tweet组件）

还要注意的另一件事是bundler自动将ClientComponent和Tweet放入两个单独的bundles中。这使得浏览器可以推迟下载Tweet bundle

