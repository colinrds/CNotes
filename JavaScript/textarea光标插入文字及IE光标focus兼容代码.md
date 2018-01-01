## 需求分析

比如你需要在一个文本框或者可编辑框的第10个字符位置插入一个表情，这时你点击某个位置，弹出一个表情列表选择框，然后选中你需要的表情，执行的文本框focus()，把代表表情的字符插入到我们上一次聚焦的位置，高级浏览器妥妥的还是原来那个位置，可IE却是聚焦到了文本框的最前面去了。

## 解决方法

我去看了一下微博的那个输入框，发现它把当前的光标位置记录在了文本框里面，使用一个range='8'这样子的属性来记录，这样子即使我们在别的地方怎么点，下一次回到这个文本框时还能知道上次光标的位置。有了这个光标位置，在插入文本前把光标放到这个位置妥妥的。

## 代码实现

根据之前的文章，我们很容易的提取出几个代码：

### 1.插入文本
```
function insertAtCursor(myField, myValue) {
    //IE support
    if (document.selection) {
    	console.log('ie');
        myField.focus();
        sel = document.selection.createRange();
        sel.text = myValue;
    }
    //MOZILLA and others
    else if (myField.selectionStart || myField.selectionStart == '0') {
    	console.log('modern');
        var startPos = myField.selectionStart;
        var endPos = myField.selectionEnd;
        myField.value = myField.value.substring(0, startPos)
            + myValue
            + myField.value.substring(endPos, myField.value.length);
        myField.selectionStart = startPos + myValue.length;
        myField.selectionEnd = startPos + myValue.length;
    } else {
        myField.value += myValue;
    }
}
```
### 2.获取光标位置

```
function getCursorPos(input) {
    if ("selectionStart" in input && document.activeElement == input) {
        return {
            start: input.selectionStart,
            end: input.selectionEnd
        };
    }
    else if (input.createTextRange) {
        var sel = document.selection.createRange();
        if (sel.parentElement() === input) {
            var rng = input.createTextRange();
            rng.moveToBookmark(sel.getBookmark());
            for (var len = 0; rng.compareEndPoints("EndToStart", rng) > 0; rng.moveEnd("character", -1)) {
                len++;
            }
            rng.setEndPoint("StartToStart", input.createTextRange());
            for (var pos = { start: 0, end: len }; rng.compareEndPoints("EndToStart", rng) > 0; rng.moveEnd("character", -1)) {
                pos.start++;
                pos.end++;
            }
            return pos;
        }
    }
    return -1;
}
```
发现上面这段代码在ie8兼容模式中获取到的值有问题。找到另外一段代码：

```
//获取选择域位置，如果未选择便是光标位置
function getSelection(el) {
    return (
    ('selectionStart' in el && function () {
        var l = el.selectionEnd - el.selectionStart;
        return {
            start: el.selectionStart,
            end: el.selectionEnd,
            length: l,
            text: el.value.substr(el.selectionStart, l)
        };
    }) ||

    (document.selection && function () {

        el.focus();

        var r = document.selection.createRange();
        if (r === null) {
            return {
                start: 0,
                end: el.value.length,
                length: 0
            }
        }

        var re = el.createTextRange();
        var rc = re.duplicate();
        re.moveToBookmark(r.getBookmark());
        rc.setEndPoint('EndToStart', re);

        return {
            start: rc.text.length,
            end: rc.text.length + r.text.length,
            length: r.text.length,
            text: r.text
        };
    }) ||

    function () {
        return null;
    }

    )();

}
```
和这个差不多类似：

```
function getInputSelection(el) {
    var start = 0, end = 0, normalizedValue, range,
        textInputRange, len, endRange;

    if (typeof el.selectionStart == "number" && typeof el.selectionEnd == "number") {
        start = el.selectionStart;
        end = el.selectionEnd;
    } else {
        range = document.selection.createRange();

        if (range && range.parentElement() == el) {
            len = el.value.length;
            normalizedValue = el.value.replace(/\r\n/g, "\n");

            // Create a working TextRange that lives only in the input
            textInputRange = el.createTextRange();
            textInputRange.moveToBookmark(range.getBookmark());

            // Check if the start and end of the selection are at the very end
            // of the input, since moveStart/moveEnd doesn't return what we want
            // in those cases
            endRange = el.createTextRange();
            endRange.collapse(false);

            if (textInputRange.compareEndPoints("StartToEnd", endRange) > -1) {
                start = end = len;
            } else {
                start = -textInputRange.moveStart("character", -len);
                start += normalizedValue.slice(0, start).split("\n").length - 1;

                if (textInputRange.compareEndPoints("EndToEnd", endRange) > -1) {
                    end = len;
                } else {
                    end = -textInputRange.moveEnd("character", -len);
                    end += normalizedValue.slice(0, end).split("\n").length - 1;
                }
            }
        }
    }

    return {
        start: start,
        end: end
    };
}
```
### 3.设置光标位置

```
function setCursorPos(input, start, end) {
    if (arguments.length < 3) end = start;
    if ("selectionStart" in input) {
        setTimeout(function() {
            input.selectionStart = start;
            input.selectionEnd = end;
        }, 1);
    }
    else if (input.createTextRange) {
        var rng = input.createTextRange();
        rng.moveStart("character", start);
        rng.collapse();
        rng.moveEnd("character", end - start);
        rng.select();
    }
}
```
### 4.记录更新光标位置

```
function updateRange(input){
    var pos = getCursorPos(input);
    input.setAttribute('range',pos.start);
}
$("#textarea").on("keyup click",function(e) {
    updateRange(this);
})
```

### 5.点击插入文字

```
$("#button").click(function(){
    var input = $('#textarea')[0];
    var range = parseInt(input.getAttribute('range'))|| input.value.length || 0;
    setCursorPos(input,range);
    setTimeout(function(){
        insertAtCursor(input,'test');
        updateRange(input);
    },1);
})
```

注意：上面的这个setTimeout定时器是必须的，如果不加，在插入文字时获取到的光标位置时错误的，需要等待上面的执行完成才能执行，我在IE9上面就遇到这个坑。