# useState 和 useRef 的区别

- 1.使用 useState 初始化的数据更新后会触发 rerender，但是修改 useRef.current 的值不会触发 rerender

但是 useRef 可以存储一些需要频繁更新的值，然后配合 useState 去触发 rerender，从而减少 rerender 次数，提升页面性能。

- 2. useRef 可以用于访问 React 组件和 DOM，useState 只能用于初始化并更新组件数据

- 3.useRef 接受一个参数（初始值），并返回一个对象，可以通过.current 属性访问初始值；useState 接受一个参数（初始值）返回一个数组，数组第一项是初始值，第二项是更新函数
