### `只贴代码 不解释过程 勿喷` 

[博客 文章地址](http://blog.ncgame.cc/article/5da7ce99abd7ed504aaa13e0);  
[github地址](https://github.com/wuxinweb/simple-virtual-dom)  
[模板](https://github.com/cvgellhorn/webpack-boilerplate.git)  

## 环境搭建
1.克隆
```
$ git clone https://github.com/cvgellhorn/webpack-boilerplate.git
$ npm install 
$ npm install @babel/plugin-transform-react-jsx --save-dev
```
2.配置babel
```
"plugins": [
    ...其他配置
    [
        "@babel/plugin-transform-react-jsx",
        {
            "pragma": "dom" // 这里表示生成的jsx 函数
        }
    ]
    ...其他配置
]
```

## 代码 `index.js`

```JavaScript

/*
 * @Author: bucai
 * @Date: 2019-10-16 20:54:45
 * @LastEditors: bucai
 * @LastEditTime: 2019-10-17 10:11:38
 * @Description: vdom
 */

// 生成的虚拟dom的节点 
function dom(type, props, ...children) {
  return {
    type,
    props,
    children
  }
}

/**
 * 生成真实的dom树
 * @param {object} domObj node节点
 */
function generateDom(domObj) {
  let $el;
  if (domObj.type) {
    $el = document.createElement(domObj.type);
  } else {
    $el = document.createTextNode(domObj);
  }

  if (domObj.props) {
    Object.keys(domObj.props).forEach(key => {
      $el.setAttribute(key, domObj.props[key]);
    });
  }

  if (domObj.children) {
    domObj.children.forEach(child => {
      $el.appendChild(generateDom(child));
    });
  }

  return $el;
}
/**
 * 对比节点是否改变
 * @param {object} nodea 节点A
 * @param {object} nodeb 节点B
 */
// node 发生改变
function isNodeChange(nodea, nodeb) {
  // 如果nodea.type 都不为空表示是 元素节点
  if (nodea.type !== undefined && nodeb.type !== undefined) {
    return nodea.type !== nodeb.type;
  }
  // 如果是其中又一个是文本节点就判断字符串 
  return nodea !== nodeb;
}
/**
 * 是否参数改变
 * @param {object} propa attrlist
 * @param {*} propb attrlist
 */
// props change
function isPropsChange(propa, propb) {
  // 统一化
  const oldProps = propa || {};
  const newProps = propb || {};
  // 获取参数的key
  const oldKeys = Object.keys(oldProps);
  const newKeys = Object.keys(newProps);
  // 长度不同就说明改变了
  if (oldKeys.length !== newKeys.length) {
    return true;
  }
  // 当长度一致的时候
  if (oldKeys.length === 0) {
    return false;
  }
  // 遍历key 来对比是否改变
  for (let i = 0; i < oldKeys.length; i++) {
    const oldkey = oldKeys[i];
    const newkey = newKeys[i];
    // 对key进行遍历
    // if (oldkey !== newkey) { // // 这里没有意义
    //   return true;
    // }
    if (oldProps[oldkey] !== newProps[newkey]) {
      return true;
    }
  }
  // 如果上面都不符合就说是没有更改的
  return false;
}

/**
 *  虚拟DOM对比函数
 * @param {HTMLElement} $parent 父节点
 * @param {object} oldNode 旧的节点对象
 * @param {object} newNode 新的节点对象
 * @param {number} index 子节点的下标
 */
function vDom($parent, oldNode, newNode, index = 0) {
  const $currlenEl = $parent.childNodes[index];
  // 旧的不存在就直接添加到dom中
  if (!oldNode) {
    // append
    return $parent.appendChild(generateDom(newNode));
  }
  // 新的不存在就直接移除旧的节点
  if (!newNode) {
    // REMOVE
    return $parent.removeChild($currlenEl);
  }
  // 判断节点是否改变
  // node change
  if (isNodeChange(oldNode, newNode)) {
    return $parent.replaceChild(generateDom(newNode), $currlenEl);
  }
  // 节点相同就没问题
  // no change textNode
  if (oldNode === newNode) {
    return;
  }

  // change props
  const oldProps = oldNode.props || {};
  const newProps = newNode.props || {};

  if (isPropsChange(oldProps, newProps)) {
    const oldPropsKey = Object.keys(oldProps);
    const newPropsKey = Object.keys(newProps);

    // 如果新的props 为空就将old全部移除
    if (newPropsKey.length === 0) {
      oldPropsKey.forEach(key => {
        $currlenEl.removeAttribute(key);
      });
    } else {

      const propsKeySet = new Set([...oldPropsKey, ...newPropsKey]);

      propsKeySet.forEach(key => {

        if (oldProps[key] === undefined) {
          $currlenEl.setAttribute(key, newProps[key]);
        } else if (newProps[key] === undefined) {
          $currlenEl.removeAttribute(key);
        } else if (newProps[key] !== oldProps[key]) {
          $currlenEl.setAttribute(key, newProps[key]);
        }

      });
    }
  }
  // 子节点
  // children change
  const oldChildren = oldNode.children || [];
  const newChildren = newNode.children || [];

  if (oldChildren.length || newChildren.length) {
    const maxLen = Math.max(oldChildren.length, newChildren.length);
    for (let i = 0; i < maxLen; i++) {
      vDom($currlenEl, oldChildren[i], newChildren[i], i);
    }
  }

}

// 原来的dom树
const profile = (
  <div>
    <h3 class="aaaa">123</h3>
    <p>123</p>
  </div>
);
// 新的dom树
const profileChange = (
  <div class="b">
    <h3 class="bucai" data-user-id="{a:1}">222</h3>
    <span>xxxxx</span>
  </div>
);
// 先生成一个真实的dom树 并将他添加到html中
const $el = generateDom(profile);
const $app = document.querySelector('#app');
$app.appendChild($el);

// 延时，方便查看
setTimeout(() => {
  // 调用虚拟dom 对比两棵树
  vDom($app, profile, profileChange);
}, 3000);


```
