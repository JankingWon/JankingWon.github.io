---
2020-04-12 18:16
---

# 干货总结！从源码分析点击事件、触摸事件、enabled、clickable的关系

## 直接来总结

附上本人总结的表格（已用代码验证）

| enabled | clickable | onTouchListener是否会被调用 | onTouchListener#onTouch返回值 | onClickListener是否会被调用 | onTouchEvent返回值 | dispatchTouchEvent返回值 |
| ------- | --------- | --------------------------- | ----------------------------- | --------------------------- | ------------------ | ------------------------ |
| true    | true      | 是                          | true                          | 否                          | true               | true                     |
| true    | true      | 是                          | false                         | 是                          | true               | true                     |
| true    | false     | 是                          | true                          | 否                          | true               | true                     |
| true    | false     | 是                          | false                         | 否                          | false              | false                    |
| false   | true      | 否                          | -                             | 否                          | true               | true                     |
| false   | false     | 否                          | -                             | 否                          | false              | false                    |

* 只要`view`是`enabled`，`onTouchListener`就会被调用
* 只有`view`是`enabled`且可点击（`clickable`、`longClickable`、`contextClickable`任意一种），且`onTouchListener`返回`false`，`onClickListener`才会被调用
* 如果`onTouchListener`没有消耗事件，且是可点击的（三种情况），那么无论设不设置`onClickListener`也无论其返回值是什么，此事件都会被消耗，因为`View`的`onTouchEvent`对事件进行了默认处理
> 不设置`onTouchListener`或`onClickListener`相当于返回`false`
> `dispatchTouchEvent`返回值是为了分析父子`View`事件分发，表示触摸事件是否被消费
## 分析

触摸事件（TouchEvent）分发的起始点为`Activity#dispatchTouchEvent(MotionEvent event)`，上一层就是Native层了

```java
public boolean dispatchTouchEvent(MotionEvent ev) {
    if (ev.getAction() == MotionEvent.ACTION_DOWN) {
        onUserInteraction();
    }
    //先给Window处理，里面再给DecorView处理，再是ViewGroup，然后分发给各个View或ViewGroup
    if (getWindow().superDispatchTouchEvent(ev)) {
        return true;
    }
    //如果没有人处理，最后自身的onTouchEvent作为兜底事件
    return onTouchEvent(ev);
}
```
### onTouchListener与onTouchEvent以及enable的影响

在`View#dispatchTouchEvent(MotionEvent event)`中有如下代码
```java
//noinspection SimplifiableIfStatement
ListenerInfo li = mListenerInfo;
if (li != null && li.mOnTouchListener != null
        && (mViewFlags & ENABLED_MASK) == ENABLED
        && li.mOnTouchListener.onTouch(this, event)) {
    //如果view是enabled，才会调用mOnTouchListener
    result = true;
}
 //如果调用mOnTouchListener的onTouch返回true，onTouchEvent不会被调用
if (!result && onTouchEvent(event)) {
    //如果view不是enabled或者mOnTouchListener的onTouch返回false，onTouchEvent被调用
    result = true;
}
```
**小结**

`onTouchEvent`作为**兜底的触摸事件处理操作**，无论`view`是不是`enabled`，此处都会调用，
但~~ 是，如果有`mOnTouchListener`并且其`onTouch`返回`true` 则表示该触摸事件已经被处理完了，则不会调用`onTouchEvent`

### onTouchEvent与onClickListener以及clickable的影响

在`View#onTouchEvent(MotionEvent event) `中有如下代码
```java
//clickable、longClickable、contenxtClickable都视为可点击
final boolean clickable = ((viewFlags & CLICKABLE) == CLICKABLE
        || (viewFlags & LONG_CLICKABLE) == LONG_CLICKABLE)
        || (viewFlags & CONTEXT_CLICKABLE) == CONTEXT_CLICKABLE;
if ((viewFlags & ENABLED_MASK) == DISABLED) {
    ......
    // 如果view不是enabled的
    // 如果是可点击的（上述三种情况），那么返回true，表示该事件被消费，但是不会真的响应它（不会调用onClickListener）；
    //如果不可点击，那么返回false，表示该事件没有被消费
    return clickable;
}
......
if (clickable || (viewFlags & TOOLTIP) == TOOLTIP) {
    switch (action) {
        case MotionEvent.ACTION_UP:
            //当事件为UP时才是点击
            mPrivateFlags3 &= ~PFLAG3_FINGER_DOWN;
            if ((viewFlags & TOOLTIP) == TOOLTIP) {
                handleTooltipUp();
            }
            //如果view不可点击（三种情况），那么就会直接返回false
            if (!clickable) {
                removeTapCallback();
                removeLongPressCallback();
                mInContextButtonPress = false;
                mHasPerformedLongPress = false;
                mIgnoreNextUpEvent = false;
                break;
            }
            //如果view是clickable执行到这里
            boolean prepressed = (mPrivateFlags & PFLAG_PREPRESSED) != 0;
            if ((mPrivateFlags & PFLAG_PRESSED) != 0 || prepressed) {
                ......
                if (!mHasPerformedLongPress && !mIgnoreNextUpEvent) {
                    ......
                    if (!focusTaken) {
                        //使用一个Runnable的对象post到队列中来执行点击事件，其内部就是执行performClickInternal()方法
                        //这是为了在单击操作之前，视图的其他视觉状态会更新（点击效果）
                        if (mPerformClick == null) {
                            mPerformClick = new PerformClick();
                        }
                        if (!post(mPerformClick)) {
                            performClickInternal();
                        }
                    }
                }
                ......
            }
            mIgnoreNextUpEvent = false;
            break;
        case MotionEvent.ACTION_DOWN:
            ......
            break;
        case MotionEvent.ACTION_CANCEL:
            ......
            break;
        case MotionEvent.ACTION_MOVE:
            ......
            break;
    }
    return true;
}
return false;
```
而`View#performClickInternal()`又调用了`View#performClick()`方法，其部分代码如下
```java
final boolean result;
final ListenerInfo li = mListenerInfo;
if (li != null && li.mOnClickListener != null) {
    playSoundEffect(SoundEffectConstants.CLICK);
    //如果mOnClickListener存在就调用其onClick方法，返回true，表示点击事件被消费
    //但是这里的返回几乎没啥用，没有人使用它的返回值
    li.mOnClickListener.onClick(this);
    result = true;
} else {
    result = false;
}
```
**小结**

* 如果`view`不是`enabled`的，那么不会调用`onClickListener`，`view`是否可点击（`clickable`、`longClickable`、`contextClickable`）**仅仅作为返回结果**表示触摸事件是否被消费
* 如果`view`是`enabled`的
    * 如果`view`可点击，那就会调用其`onClickListener`，返回`true`表示触摸事件被点击监听器消费了
    * 如果`view`不可点击，直接返回`false`，表示触摸事件没有被消费

> 如果纰漏，欢迎指正