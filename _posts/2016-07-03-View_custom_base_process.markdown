---
layout: post
title:  "自定义View流程梳理"
date:   2016-07-02 15:57:54
categories: tools
---

### 流程概要

绘制一个View基本分为*测量*和*绘制*两个主要方面，分别确定了视图的大小和内容。

---

### 构造函数

一般视图有如下几种构造函数：

	public void MyView(Context context); // 通过Java代码构造视图时使用该版本
	public void MyView(Context context, AttributeSet attrs); // 从XML填充时使用该版本。AttributeSet包括附加到视图的XML元素的所有属性
	public void SloopView(Context context, AttributeSet attrs, int defStyleAttr); // 类似上一方法，增添了样式属性
	
通常将上述几个构造方法连在一起使用，例如：

	public MyView(Context context){
		this(context, null);
	} 
	public MyView(Context context, AttributeSet attrs){
		this(context, attrs, 0);
	}
	public MyView(Context context, AttributeSet attrs, int defStyle){
		super(context, attrs, defStyle);
		// 在这里做你要做的初始化
	}

---

### onMeasure方法

View必须实现onMeasure方法来向框架提供其内容的测量，并在该方法返回前调用setMeasureDimension()方法设定测量的宽高。另外，在测量期间，视图实际上还没有确定的大小，它只有已测量的尺寸。如果在分配大小后需要对视图做一些自定义工作，则应该重写onSizeChanged()并添加适当的代码。

关于onMeasure(int widthMeasureSpec, int heightMeasureSpec)的参数widthMeasureSpec和heightMeasureSpec包含了宽、高和各自方向上对应的测量模式信息。要获取宽高及其测量模式可调用如下方法：

	int width = MeasureSpect.getSize(widthMeasureSpec);
	int widthMode = MeasureSpec.getMode(widthMeasureSpec);
	int height = MeasureSpect.getSize(heightMeasureSpec);
	int heightMode = MeasureSpec.getMode(heightMeasureSpec);

测量模式有三种：

|模式|二进制数|描述|
|:-:|:-:|:-:|
|MeasureSpect.AT_MOST|10|属性为match_parent或有其他大小上限；在模式下报告的大小不应超过规范中规定的值|
|MeasureSpect.EXACTLY|01|参数为固定值|
|MeasureSpect.UNSPECIFIED|00|无约束或是wrap_content;可报告默认大小|

---

### onLayout()

在定制GroupView时调用该方法来确定子View的位置。

---

### onDraw()

在这里进行画布的绘制操作。

---

### Demo

附上一个饼状图的demo，仅做流程示意用：

	package com.softcloud.testa5.widgets;

	import android.annotation.TargetApi;
	import android.content.Context;
	import android.graphics.Canvas;
	import android.graphics.Color;
	import android.graphics.Paint;
	import android.graphics.Point;
	import android.os.Build;
	import android.util.AttributeSet;
	import android.view.View;

	import com.softcloud.testa5.R;

	import java.util.ArrayList;
	import java.util.List;

	/**
 	* Created by Softcloud on 16/6/14.
 	*/	
	public class PieChat extends View {

    private final int[] colorResIds = {
            R.color.red, R.color.green, R.color.blue, R.color.purple, R.color.colorPrimary,
            R.color.pink, R.color.colorPrimaryDark, R.color.colorAccent, R.color.orange
    };

    private Point center;
    private int radius;
    private final int DEFAULT_WIDTH = 200;
    private Paint paint;

    private List<Double> dataList = new ArrayList<>();
    private double[] dataProportion;

    public PieChat(Context context) {
        this(context, null);
    }

    public PieChat(Context context, AttributeSet attrs) {
        this(context, attrs, 0);
    }

    public PieChat(Context context, AttributeSet attrs, int defStyleAttr) {
        super(context, attrs, defStyleAttr);
        center = new Point();
        paint = new Paint(Paint.ANTI_ALIAS_FLAG);
        paint.setStyle(Paint.Style.FILL);
    }

    @Override
    protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
        int width, height;
        width = getRealWidthFromMeasureSpec(widthMeasureSpec);
        height = getRealWidthFromMeasureSpec(heightMeasureSpec);
        setMeasuredDimension(width, height);
    }

    private int getRealWidthFromMeasureSpec(int measureSpec) {
        int specSize = MeasureSpec.getSize(measureSpec);
        switch (measureSpec) {
            case MeasureSpec.AT_MOST:
                return Math.min(specSize, DEFAULT_WIDTH);
            case MeasureSpec.EXACTLY:
                return specSize;
            case MeasureSpec.UNSPECIFIED:
                return DEFAULT_WIDTH;
            default:
                return specSize;
        }
    }

    @Override
    protected void onSizeChanged(int w, int h, int oldw, int oldh) {
        if (w != oldw || oldh != h) {
            center.x = w / 2;
            center.y = h / 2;
            radius = Math.min(center.x, center.y);
        }
    }

    @TargetApi(Build.VERSION_CODES.LOLLIPOP)
    @Override
    protected void onDraw(Canvas canvas) {
        canvas.translate(center.x, center.y);
        canvas.rotate(-90);
        if (dataList.isEmpty()) {
            paint.setColor(Color.BLACK);
            canvas.drawCircle(0, 0, radius, paint);
        } else {
            calculatePropotions();
            int partCount = dataProportion.length;
            float startAngle = 0;
            float sweepAngle = 0;
            float endAngle = 0;
            for (int i = 0; i < partCount - 1; i++) {
                paint.setColor(getResources().getColor(colorResIds[i]));
                sweepAngle = (float) (dataProportion[i] * 360);
                canvas.drawArc(-radius, -radius, radius, radius, startAngle, sweepAngle,
                        true, paint);
                startAngle += sweepAngle;
            }
            paint.setColor(getResources().getColor(colorResIds[dataProportion.length - 1]));
            endAngle = 360;
            canvas.drawArc(-radius, -radius, radius, radius, startAngle, endAngle - startAngle,
                    true, paint);
        }
    }

    private void calculatePropotions() {
        double dataTotal = getDataTotal();
        dataProportion = new double[dataList.size()];
        for (int i = 0; i < dataList.size(); i++) {
            dataProportion[i] = dataList.get(i) / dataTotal;
        }
    }

    private double getDataTotal() {
        int total = 0;
        for (Double data : dataList) {
            total += data;
        }
        return total;
    }

    public boolean addData(double data) {
        if (data > 0 && dataList.size() < colorResIds.length) {
            dataList.add(data);
            invalidate();
            return true;
        } else {
            return false;
        }
    }
}

