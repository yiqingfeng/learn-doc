# 事件池、事件驱动
----

```javascript
/*
 * 事件池
 * 说明：结合Backbone.Events和Phalcon中的EventManager
 * 作者：Mengxuan
 */
;(function (root){
    var eventsMap = {},
        eventBus = {};
    root.eventBus = eventBus;
    // 绑定事件
    eventBus.bind = eventBus.on = function (name, callback, context){
        var me = this;
        if (!EventsApi(me, 'on', name, [callback, context]) || !callback) return me;

        // 使用关联数组记录注册（绑定）的事件
        var events = eventsMap[name] || (eventsMap[name] = []);
        events.push({
            callback: callback,
            cxt: context || me
        });
        return me;
    }
    // 解绑事件
    eventBus.unbind = eventBus.off = function (name, callback, context){
        var me = this;

        // 无参时，取消所有事件的绑定
        if (!name) {
            eventsMap = {};
            return me;
        }

        if (!EventsApi(me, 'off', name, [callback, context])) return me;
        if (!eventsMap[name]) return me;

        var events = eventsMap[name];
        // 移除该事件对应的所有事件
        if (!callback) {
            delete eventsMap[name];
            return me;
        }

        // 通过新数组来保存剩余的回调序列
        var remaining = [];
        context || (context = me);
        for (var i=0,l=events.length; i<l; i++){
            var event = events[i];
            if (callback !== event.callback || context !== event.cxt) {
                remaining.push(event);
            }
        }
        if (remaining.length) {
            eventsMap[name] = remaining;
        } else {
            delete eventsMap[name];
        }
        return me;
    }
    // 触发事件, 传入的data为对象
    eventBus.fire = eventBus.trigger = function (name, data, context){
        var me = this;
        if (!EventsApi(me, 'trigger', name, [data])) return me;

        // 允许二级事件，即"change:before"
        var event = eventsMap[name],
            secEventSeparator = /\:/,
            names = name.split(secEventSeparator),
            firEvent = eventsMap[names[0]];
        context || (context = me);
        if (names.length > 1 && firEvent) {
            // 执行一级事件序列，可是为初始化该类型事件
            triggerEvents(firEvent, data);
        }
        if (event) {
            triggerEvents(event, data);
        }
        return me;
    }
    // 绑定一个事件，只执行一次
    eventBus.once = function (name, callback, context){
        var me = this;
        if (!EventsApi(me, 'once', name, [callback, context]) || !callback) return me;

        // 使用关联数组记录注册（绑定）的事件
        var events = eventsMap[name] || (eventsMap[name] = []),
            once = function (){
                me.off(name, once);
                callback.apply(this, arguments);
            };

        events.push({
            callback: once,
            cxt: context || me
        });
        return me;
    }
    // 执行事件序列
    function triggerEvents(callbacks, data){
        for (var i=0,l=callbacks.length; i<l; i++) {
            callbacks[i].callback.call(callbacks[i].cxt, data);
        }
    }
    // 实现多事件绑定的事件API，即"change blur"
    function EventsApi(obj, action, name, args) {
        // 多事件以空格相区分
        var eventSeparator = /\s+/;

        if (!name) return true;

        if (eventSeparator.test(name)) {
            var names = name.split(eventSeparator);
            for (var i=0,l=names.length; i<l; i++) {
                obj[action].apply(obj, [names[i]].concat(args));
                console.log(names[i]);
            }
            return false;
        }
        return true;
    }
})(window);
```