
给`UIImageView`设置圆角和阴影

```
UIImageView *imgView = [UIImageView new]
imgView.layer.cornerRadius = 5;
imgView.layer.shadowPath = ..

```

这样设置之后，该控件有了阴影，但是显示图片时并没有圆角。

因为显示图片时，图片是以`UIImageView`的bounds来显示的，而裁剪的是layer。

但是如果通过`maskToBounds`属性的话，就会将阴影裁剪掉。

所以，只能通过`CoreGraphics`将图片裁剪成圆形，然后放到这个`UIImageView`上面来。

