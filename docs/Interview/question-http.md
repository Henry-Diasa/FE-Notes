#### 重写xhr

```js
class XhrHook {
	constructor(beforeHooks = {}, afterHooks = {}) {
		this.XHR = window.XMLHttpRequest
		this.beforeHooks = beforeHooks
		this.afterHooks = afterHooks
		this.init()
	}
	init() {
		let _this = this
		window.XMLHttpRequest = function() {
			this._xhr = new _this.XHR()
			_this.overwrite(this)
		}
	}
	overwrite(proxyXHR) {
		for(let key in proxyXHR._xhr) {
			if(typeof proxyXHR._xhr[key] === 'function') {
				this.overwriteMethod(key, proxyXHR)
				continue
			}
			this.overwriteAttributes(key, proxyXHR)
		}
	}
	overwriteMethod(key, proxyXHR) {
		let beforeHooks = this.beforeHooks
		let afterHooks = this.afterHooks
		proxyXHR[key] = (...args) => {
			if(beforeHooks[key]) {
				const res = beforeHooks[key].call(proxyXHR, args)
				if(res === false) {
					return 
				}
			}
			
			const res = proxyXHR._xhr[key].apply(proxyXHR._xhr, args)
			afterHooks[key] && afterHooks[key].call(proxyXHR._xhr, res)
			return res
		}
	}
	overwriteAttributes(key, proxyXHR) {
		Object.defineProperties(proxyXHR, key, this.setPropertyDescriptor(key, proxyXHR))
	}
	setPropertyDescriptor(key, proxyXHR) {
		let obj = Object.create(null)
		let _this = this
		
		obj.set = function(val) {
			if(!key.startsWith('on')) {
				proxyXHR['_' + key] = val
				return
			}
			
			if(_this.beforeHooks[key]) {
				this._xhr[key] = function (...args) {
					_this.beforeHooks[key].call(proxyXHR)
					val.apply(proxyXHR, args)
				}
				return
			}
			this._xhr[key] = val
		}
		
		obj.get = function() {
			return proxyXHR['_' + key] || this._xhr[key]
		}
		return obj
	}
}
```

