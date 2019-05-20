# UITextField常见问题

当使用`UITextfield`需要考虑的可能会发生问题的点

1. 明密文切换问题

2. 复制粘贴时不会触发通知的问题

## UITextField直接对text属性进行赋值时的检查点

1. 检查是否有监听`UITextFieldTextDidChangeNotification`通知

当在`UITextField`的代理方法`shouldChangeCharactersInRange:replacementString`中直接对`textField.text`赋值时，一定要检查是否有监听这个`textField`的`UITextFieldTextDidChangeNotification`通知。

因为直接赋值是不会走这个通知的。

#### 场景还原

遇到这个问题是在`UITextField`明文和密文切换时

当明文密文切换之后，点击退格键，会直接将所有内容删除掉。

为了解决该问题就需要，判断如果是密文状态，通过赋值的方式更改`textField`的内容。

因为赋值之后没有发送通知，又导致了监听这个文本框通知没有执行的问题。

```
- (BOOL)textField:(UITextField *)textField shouldChangeCharactersInRange:(NSRange)range replacementString:(NSString *)string {
    NSString *numberString = [textField.text stringByReplacingCharactersInRange:range withStringSafely:string];
    if (textField.isSecureTextEntry==YES) {
        textField.text= numberString;
        [[NSNotificationCenter defaultCenter] postNotificationName:UITextFieldTextDidChangeNotification object:textField];
        return NO;
    }
}
```

2. 检查是否监听了`rac_textSignals`这个信号

当订阅了`rac_textSignals`信号时，直接对`textField.text`赋值时，该信号也是不会收到响应的

解决方式可以通过

```
	[[RACSignal merge:@[textField.rac_textSignals, RACObserve(textField, text)]] subscribeNext:]
```

## UITextField 明文密文切换之后有可能出现以下情况

1. 字体改变 

2. 光标位置改变

3. 密文-明文-密文，切换之后再输入之前的内容被清空

### 字体改变问题 + 光标位置改变问题

这两个问题都可以在点击明文密文切换按钮时来进行处理

当**点击明文密文切换的按钮**时，重新设置一下字体，**解决字体自动更改问题**。

当**点击明文密文切换按钮**时，重新设置一下文本，**解决光标位置问题**

解决方案如下：

```
- (void)changeSecuire {
    self.textField.secureTextEntity = !self.textField.secureTextEntity;
    btn.selected = !self.secureTextEntry;

    // 先获取原来的文本，再重新赋值，解决光标问题
    NSString *originText = self.text;
    self.text = @"";
    self.text = originText;

    // 重新设置一下字体，解决字体自动改变问题
    self.font = nil;
    self.font = [UIFont boldSystemFontOfSize:18.0];

}
```
