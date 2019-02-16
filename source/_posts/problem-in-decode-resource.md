title: BitmapFactory.decodeResource 原来只能加载「图片格式」的资源啊……
date: 2017-05-03 10:00:00
categories: 技术
tags: Android
description: 啊，是真的
---

今天在用公司提供的网络图片加载控件进行开发的时候，因为图片上有个蒙层的需求，所以很自然的调用了这个图片加载控件的 `setOverlay(int resId)`的方法，用来加载一个事先定义在`.xml`中的方形 Shape。

代码很简单，吭哧吭哧的写好之后插电运行——啪，崩崩崩。顿时懵逼了，不至于吧，写个这么简单的数据绑定逻辑都能蹦蹦蹦？赶紧拉报错堆栈来看，唔，一个NPE的错误-\>一个 Bitmap 为空-\>传入的这个 Overlay 的 Bitmap 为空-\>嗯……那就是这句话得到的 Bitmap 为空咯：

	overlay = BitmapFactory.decodeResource(getContext().getResources(), overlayResId, options)
	
我凭实力传入的ResId凭什么解析不出Bitmap！但是它还真的就不会返回 Bitmap，我们看看他的代码段：

```java
	/**
	     * Synonym for opening the given resource and calling
	     * {@link #decodeResourceStream}.
	     *
	     * @param res   The resources object containing the image data
	     * @param id The resource id of the image data
	     * @param opts null-ok; Options that control downsampling and whether the
	     *             image should be completely decoded, or just is size returned.
	     * @return The decoded bitmap, or null if the image data could not be
	     *         decoded, or, if opts is non-null, if opts requested only the
	     *         size be returned (in opts.outWidth and opts.outHeight)
	     */
	    public static Bitmap decodeResource(Resources res, int id, Options opts) {
	        Bitmap bm = null;
	        InputStream is = null; 
	        
	        try {
	            final TypedValue value = new TypedValue();
	            is = res.openRawResource(id, value);
	
	            bm = decodeResourceStream(res, value, is, null, opts);
	        } catch (Exception e) {
	            /*  do nothing.
	                If the exception happened on open, bm will be null.
	                If it happened on close, bm is still valid.
	            */
	        } finally {
	            try {
	                if (is != null) is.close();
	            } catch (IOException e) {
	                // Ignore
	            }
	        }
	
	        if (bm == null && opts != null && opts.inBitmap != null) {
	            throw new IllegalArgumentException("Problem decoding into existing bitmap");
	        }
	
	        return bm;
	    }
	    
```

确实啊，人家白纸黑字的写了，这个方法要么返回一个从原始图片文件得到的 Bitmap，否则返回 null，而且传入的 Option 参数也是对应的类似采样率这样的图片编码相关的需求——这段代码的的确确只能用来将「**原始的图片文件**」加载为 Bitmap，是我用错了资源类型，锅在我自己脑袋上。

但是光是找到问题还是不够的，得好好想想这个问题：为什么我会在一开始看不出这个问题——对 Bitmap 和 Drawable 概念的混淆是我没能及时定位这个问题的最大原因。Bitmap 是一个比 Drawable 更加底层的概念，它对应的是每个像素点在内存里的数据，而 Drawable 则是一个抽象的概念，他是 Bitmap 的超集，一个Drawable 对象可能来自于 xml 定义、来源于 .jpg/.png 文件、来源于 Bitmap。

但是这里最大的锅还是在平台——平台提供的`setOverlay(int resId)`方法显然并没有对于这些不同的 Res 作区分，默认其均为 .jpg/.png 文件的资源，并使用加载 Bitmap 的方法去加载这些资源，而调用者对于此一无所知，既没有方法名上的提醒，也没有文档的提醒，自然在使用时会出现问题。

这里还是总结一下两个经验教训吧：1. 对于 Bitmap 和 Drawable 的基础概念需要有明确的辨识；2. 对于 API 设计需要考虑调用者的使用情景，不能想当然的给出方法，如果确实如此，则需要给出详细的文档作参考。