---
title: vue2.0双向绑定浅析
date: 2020-3-2 14:37:07
updated: 2020-5-11 20:11:58
tags: 
- Vue 
categories: Javascript
---

Vue是前端三大框架之一，它的小而精获得国内外诸多开发者的青睐。对于Vue的核心功能双向绑定的实现是初中级开发者非常关注的难点

## 关键词
观察者，订阅者，消息接收器

## 思路
* 数据驱动视图更新
* 视图驱动数据更新

## 实现过程

> 1、初始化数据

    <div id="app">
      <input type="text" v-model="name">
      <div>{{name}}</div><br>
      <input type="text" v-model="age">
      {{age}}
    </div>

    let vm = new Vue({
			el: '#app',
			data: {
				name: 'jane',
				age: 25
			}
		})

> 2、遍历data里的每一项，通过Object.defineProperty生成一个观察者模式，通过设置getter和setter控制属性的访问和更改

    observe(data) {
      if (!data || typeof data !== 'object') {
            return;
        }
        Object.keys(data).forEach((key) => {
            this.defineReactive(data, key, data[key]);
        });
    }
    defineReactive(data, key, val) {
        this.observe(val); //递归遍历所有子属性
        var dep = new Dep(); //为每一个属性新建消息接收器
        Object.defineProperty(data, key, {
            enumerable: true,
            configurable: true,
            get() {
                if (Dep.target) {
                  console.log('添加成功')
                    dep.addSub(Dep.target); // 添加订阅者到消息接收器
                }
                return val;
            },
            set(newVal) {
                if (val === newVal) {
                    return;
                }
                val = newVal;
                console.log('现在属性值为：“' + newVal.toString() + '”');
                dep.notify(); // 如果数据变化，通知订阅者
            }
        });
    }

> 3、 遍历DOM树，并为每一个绑定值生成订阅者

    childNodes(el) {
      [...el.childNodes].forEach((node) => {
        if (node.nodeType === 3) {
          const text = node.textContent
          const regexp = /\{\{\s*([^\s\{\}]+)\s*\}\}/g
          if (regexp.test(text)) {
            // console.log(RegExp.$1)
            if (this._data[RegExp.$1]) {
              node.textContent = node.textContent.replace(regexp, this._data[RegExp.$1])
              new Watcher(this, RegExp.$1, function (value) {  // 生成订阅器并绑定更新函数
                    this.updateText(node, value);
                });
            }
          }
        }
        if (node.nodeType === 1) {
          this.childNodes(node)
        }
      })
    }

> 4、 生成订阅者模式的同时触发一次getter把当前订阅者存在消息接收器dep中

    class Watcher {
      constructor(vm, exp, cb) {
      //vm为colors实例，exp为属性名，cb为渲染函数
      console.log(vm)
      this.cb = cb;
        this.vm = vm;
        this.exp = exp;
        this.value = this.get();  // 添加到订阅器
      }
      get() {
          Dep.target = this;  // 缓存自己
          var value = this.vm._data[this.exp] // 强制触发监听器里的get函数
          Dep.target = null;  // 释放自己
          return value;
      }
  }

> 5、为绑定v-model关键词的dom元素绑定input事件，通过input事件触发绑定值的更新

    let attr = node.attributes
    if (attr.hasOwnProperty('v-model')) {
      let key = attr['v-model'].nodeValue
      // console.log(key)
      if(this._data[key]) {
        node.value = this._data[key]
        node.addEventListener('input', (e) => {
          this._data[key] = node.value
        })
      }  
    }

> 6、触发setter的时候通知消息接收器更新并触发watcher的update更新view层

    class Dep {
      constructor() {
        this.subs = [];
      }
      addSub(sub) {
            this.subs.push(sub);
      }
      notify() {
          this.subs.forEach((sub) => {
              sub.update()
          });
      }
    }

    watcher中：
    update() {
        this.run();
    }
	  run() {
      var value = this.vm._data[this.exp];
      var oldVal = this.value;
      if (value !== oldVal) {
        this.value = value;
        this.cb.call(this.vm, value);
      }
    }

> 最终效果

<video  width="800" height="520" controls>
  <source src="/medias/vue1/vue1.mp4">
<video>