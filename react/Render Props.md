

# Render Props

直接从一个例子开始，点击元素，为当前元素随机改变一种背景色

```react
import React, { useState } from 'react';
const ChangeBgColor = (props) => {
    const [color, setColor] = useState('#fff');
    const randomNumber = () => {
        return Math.floor(Math.random() * 255);
    };
    const change = () => {
        setColor(`rgb(${randomNumber()},${randomNumber()},${randomNumber()})`);
        console.warn(color);
    };
    return (
        <div
            style={{
                backgroundColor: color
            }}
            onClick={change}
        >
            改变背景色
        </div>
    );
};
export default ChangeBgColor;
```

实现这个功能很简单，只需要定义个变量来存储颜色，然后为元素添加一个点击事件，点击后更新颜色。

如果很多地方都需要使用这个功能，一直copy简单的操作，会显得我们很low，有必要把这个功能抽成一个组件。

先想一下这个组件的功能是什么，是更换背景色，颜色属性与改变颜色的方法都应该在这个组件中实现。但是元素内容是变化的，所以内容是父组件传递下来的，可以使用 `props` 来接受。于是就有了下面的代码：

```react
import React, { useState } from 'react';

const ChangeBgColor = (props) => {
    const {
        render
    } = props;
    const [color, setColor] = useState('#fff');
    const randomNumber = () => {
        return Math.floor(Math.random() * 255);
    };

    const change = () => {
        setColor(`rgb(${randomNumber()},${randomNumber()},${randomNumber()})`);
    };

    return (
        <div
            style={{
                backgroundColor: color
            }}
            onClick={change}
        >
            {
                render()
            }
        </div>
    );
};

export default ChangeBgColor;
```

*使用组件：*

```react
<ChangeBgColor
    render={
        () => {
            return (
                <div>
                    改变背景色
                </div>
            );
        }
    }
/>
```

很明显这种方式存在问题，并没有真正意义上改变目标元素的背景色，只是在原来的基础上套了一层而已。如果组件内容本身存在背景色的话，连效果都看不到了。

既然 `render` 接收的是一个函数，我们可以利用 `render` 进行传参，把颜色与改变颜色方法传入 `render` 中，在函数内部直接使用不就可以了么：

```react
import React, { useState } from 'react';
const ChangeBgColor = (props) => {

    const {
        render
    } = props;

    const [color, setColor] = useState('#fff');

    const randomNumber = () => {
        return Math.floor(Math.random() * 255);
    };

    const change = () => {
        setColor(`rgb(${randomNumber()},${randomNumber()},${randomNumber()})`);
    };

    return (
        <div>
            {
                render(color, change)
            }
        </div>
    );

};

export default ChangeBgColor;
```

*使用组件：*

```react
<ChangeBgColor
    render={
        (color, change) => {
            return (
                <div
                    style={{
                        backgroundColor: color
                    }}
                    onClick={change}
                    >
                    改变背景色
                </div>
            );
        }
    }
/>
```

通过render方法接受组件内部的属性与方法来控制其行为，这样就可以达到功能与内容分离的目的，使功能可以更加灵活的被复用。

用 `props` 来接受 `render` 方法，这就是render props名字的由来。当然这里的属性名 `render` 是可以改成任意其他名字的。

不过每次使用时还是需要将内容传递给一个属性，看起来也不舒服，我们可以利用react的自带的props.children属性来传递 `render`：

```react
import React, { useState } from 'react';
const ChangeBgColor = (props) => {

    const {
        render
    } = props;

    const [color, setColor] = useState('#fff');

    const randomNumber = () => {
        return Math.floor(Math.random() * 255);
    };

    const change = () => {
        setColor(`rgb(${randomNumber()},${randomNumber()},${randomNumber()})`);
    };

    return (
        <div>
            {
                render(color, change)
            }
        </div>
    );

};

export default ChangeBgColor;
```

*使用组件：*

```react
<ChangeBgColor>
    {
        (color, change) => {
            return (
                <div
                    style={{
                        backgroundColor: color
                    }}
                    onClick={change}
                >
                    改变背景色
                </div>
            );
        }
    }
</ChangeBgColor>
```

你还可以利用这些参数做一些其他的事情，比如说想显示当前的背景色的值：

```react
<ChangeBgColor>
    {
        (color, change) => {
            return (
                <Fragment >
                    <div>
                    	当前背景色：{color}
                    </div>
                    <div
                        style={{
                            backgroundColor: color
                        }}
                        onClick={change}
                    >
                        改变背景色
                    </div>
                </Fragment>
            );
        }
    }
</ChangeBgColor>
```