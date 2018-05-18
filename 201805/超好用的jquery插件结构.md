---
title: 超好用的jquery插件结构
tags: 
 - javascript
 - jquery
comments: true
date: 2018-05-13 18:34:14
---
### 场景
当你在工作中可能会遇到一些情况，你可能需要写一些小的jquery插件来满足比如模板创建、删除块、左右移动块等等需求。由于js是全局共享的语言，所以想实现上述的功能并不是什么难事，但是正也因为js是全局的语言，所以我们会遇到一个棘手的问题，是变量、方法的管理问题，怎样调用方法，怎样调用共有变量，怎么样能让代码有更好的可读性，怎么样方便其他开发者和用户的调用，都成为了我们要考虑的问题。通过阅读一些开源的js插件，他们给出了一套完整而简单、实用的js插件结构。以下我们简称为**jquery结构**。
<!--more-->
### 结构优势
1. 通过闭包，隔离了外界方法、变量的干扰
2. 设置默认变量
3. 提供用户外部属性配置
4. 提供用户外部调用结构中有无属性的方法
5. 与jquery结合良好，可以通过jquery方式在全局访问具体的插件实现
6. 插件与插件之间可以相互调用，扩展性好
7. 结构中有静态变量、实例变量，方便不同的用途
8. 插件内部的方法能相互调用
9. 调用插件后能返回处理后的jquery对象结构，方便用户做进一步的调用或者展示
### 代码实例
```
/*
 *	景区别名编辑器
 *	@author yknife 
 *	@date 2018/01/03
 */
;(function($){
	
	var ActivityOtherTitleEditor,mainTpl;
	
	mainTpl = "<div class=\"col-md-3 pad\" data-type=\"item\" >"+
    "<input type=\"text\" class=\"form-control other-title-item\" placeholder=\"$placeholder\" name=\"$otherTitleName\"/>"+
        "<span class=\"edgeBtn\">"+
            "<span class=\"ion ion-ios-close-outline\"></span>"+
        "</span>"+
    "</div>";                                                                                                                    
	
	/*
	 * 插件类
	 * @param element DOM元素
	 * @param options 初始化用户配置参数
	 * */
	ActivityOtherTitleEditor = function(element,options){
		var self = this;
		//jquery对象
		self.$element = $(element);
		self.options = options;//插件加载用户配置项
	}
	
	/*
	 * 插件方法原型
	 * 
	 * */
	ActivityOtherTitleEditor.prototype = {
		//设置构造函数
		construtor:ActivityOtherTitleEditor,
		//初始化
		init:function(data){
			var self = this;
			var titleArr = data.split(",");
			for(var i=0;i<titleArr.length;i++){
				var obj = self.add();
				self.setValue(obj,titleArr[i]);
			}
		},
		getCount:function(){
			var self = this;
			var el = self.$element;
			return el.find("div[data-type='item']").length;
		},
		add:function(){
			var self = this;
			if(self.getCount()>=self.options.maxLength){
				$.dialog.notify("最多只能添加"+self.options.maxLength+"个");
				return;
			}
			var newAdd = new String(mainTpl);
			newAdd = newAdd.replace(/\$otherTitleName/g,"otherTitleName_"+self.uuid());
			if(self.options.placeholder)
				newAdd = newAdd.replace(/\$placeholder/g,self.options.placeholder);
			else
				newAdd = newAdd.replace(/\$placeholder/g,"别名");
			var newObj = $(newAdd);
			self.$element.append(newObj);
			newObj.find(".edgeBtn").click(function(){
				self.onRemoveRow(newObj);
			});
			return newObj;
		},
		setValue:function(obj,data){
			$(obj).find("input").val(data);
		},
		getValues:function(){
			var self = this;
			var el = self.$element;
			var arr = [];
			el.find(".other-title-item").each(function(){
				var otherTitle = $(this).val();
				var i = arr.indexOf(otherTitle);
				if(otherTitle&&i==-1)
					arr.push(otherTitle);
			})
			var str = arr.join(",");
			return str;
		},
		onRemoveRow:function(obj){
			var self = this;
			obj.remove();
		},
		uuid:function(){
			var s = [];
		    var hexDigits = "0123456789abcdef";
		    for (var i = 0; i < 36; i++) {
		        s[i] = hexDigits.substr(Math.floor(Math.random() * 0x10), 1);
		    }
		    s[14] = "4";  // bits 12-15 of the time_hi_and_version field to 0010
		    s[19] = hexDigits.substr((s[19] & 0x3) | 0x8, 1);  // bits 6-7 of the clock_seq_hi_and_reserved to 01
		    s[8] = s[13] = s[18] = s[23] = "-";
		 
		    var uuid = s.join("");
		    return uuid;
		}
	}
	
	/*
	 * 插件动态方法调用
	 * */
	$.fn.ActivityOtherTitleEditor = function(option){
		
		//为了能兼容多出arguments格式
		var args = Array.apply(null,arguments);
		//返回对象
		var retvals = [];
		//移除第一个元素
		args.shift();
		//初始化每一个元素
		this.each(function(){
			//
			var self = $(this);
			//已经绑定到DOM元素上的插件数据
			var data = self.data("activityOtherTitleEditor");
			//初始化的时候返回用户配置
			var options = typeof option === "object" && option;
			//没有绑定
			if(!data){
				//合并配置
				var opts = $.extend(true,{},$.fn.ActivityOtherTitleEditor.defaults,options);
				//初始化
				data = new ActivityOtherTitleEditor(this,opts);
				//绑定DOM和数据
				self.data("activityOtherTitleEditor",data);
			}
			//调用插件方法
			if(typeof option === "string"){
				//执行每个元素的方法
				retvals.push(data[option].apply(data,args));
			}
		})
		
		switch (retvals.length) {
		//例如：初始化时
		case 0:
			return this;
		//例如：$("#id")
		case 1:
			return retvals[0];
		//例如： $(".class")
		default:
			return retvals;
		}
	}
	
	/*
	 * 插件默认参数配置
	 * */
	$.fn.ActivityOtherTitleEditor.defaults = {
		//最多能添加个数
		maxLength : 10
	}
	
})(jQuery);
```
### 代码拆解
因为我经常使用的代码是java，所以当我遇得到一个新语言的时候，我会先入为主的将java上的一些特性与新语言的上的特性做一个映射，如果新语言没有这种特性，新语言应该也有什么方式能实现java上的特性，你可以去找到，这样能比较轻松快速的学会一些新的语言，而且不会有什么遗漏，记得牢。我们现在对上面的**代码实例**做一个拆解。
* **变量**。当我们新接触一个语言的时候，我们第一时间想用的就是这个语言的变量，常用的变量会有成员变量、静态变量、局部变量，我们这里看看jquery结构是怎么实现这些特性，并怎么调用的。
#### 成员变量
```
//java实现
public String name;
```
```
//jquery结构现实
var self = this;
self.options = options;//self.options设置以后，可以在插件内部方法调用，且是实例唯一的，所以是成员变量
```
#### 静态变量
```
//java实现
public static String name;
```
```
//jquery结构现实
var ActivityOtherTitleEditor,mainTpl;//可以在所有方法中直接使用，且是类唯一，不随实例的改变而改变。
```
#### 局部变量
```
//java实现
public void myMethod(){
    String name;
}
```
```
//jquery结构现实
function myMethod(){
    var name;//这里没有什么太大区别
}
```
* **构造函数**。有了变量以后，你会希望初始化这些变量，这时候我们需要构造函数。
#### 构造函数
```
//java实现
public MyConstructor(String name){
    this.name = name;
}
```
```
//jquery结构现实
ActivityOtherTitleEditor = function(element,options){
	var self = this;
	self.options = options;//插件加载用户配置项
}
```
* **成员函数**。构造完实例，我们迫不及待的要开始我们的具体工作，这时候我们要用到成员函数。jquery结构的成员函数使我们无论从内部还是外部调用函数成为了可能。
#### 成员函数
```
//java实现
public void myMethod(){
}
```
```
//jquery结构实现,实现起来比java看上去要费劲许多，但是很好的将方法都归集在一起，对方法维护带来了许多便利。

ActivityOtherTitleEditor.prototype = {
	//设置构造函数
	construtor:ActivityOtherTitleEditor,
	//初始化
	init:function(data){
		var self = this;
		var titleArr = data.split(",");
	}
}
```
---
以上我们已经介绍了jquery结构的主干部分，在java中这些部分已经足够让我们编写出我们的类代码，但是jquery结构还需要一些粘合剂和一些更加健壮的配置项。
* **粘合剂、引擎**。jquery结构的这个部分对用户是透明的，编写插件的人通过这个部分实现了参数与配置项的合并，插件的实例化，将插件的业务数据绑定到DOM之中，执行外部动态调用插件成员函数，返回经过包装的jquery对象等一系列的功能，可以说是整个jquery结构的引擎，它的本质其实是动态方法调用。
#### 参数与配置项的合并
```
//合并配置
var opts = $.extend(true,{},$.fn.ActivityOtherTitleEditor.defaults,options);
```
#### 插件的实例化
```
var data = self.data("activityOtherTitleEditor");
data = new ActivityOtherTitleEditor(this,opts);
```
#### 将插件的业务数据绑定到DOM
```
//绑定DOM和数据
self.data("activityOtherTitleEditor",data);
```
#### 执行外部动态调用插件成员函数
```
retvals.push(data[option].apply(data,args));
```
#### 返回经过包装的jquery对象
```
//返回经过包装的jquery对象，可以链式调用后续的方法
switch (retvals.length) {
//例如：初始化时
case 0:
	return this;
//例如：$("#id")
case 1:
	return retvals[0];
//例如： $(".class")
default:
	return retvals;
}
```



* **插件默认参数配置**。当你查看大部分开源API的时候，你都会看到一些配置项，他们的风格大体如下({}部分)：
#### 默认参数
```
//用户配置参数
var select2 = $("#select_2").areaPicker({
	//请求数据
	url:"...",
	//上级
	parentArea:select1,
	//是否清空已选项
	allow_single_deselect:true,
	//搜索没有结果显示文字
	no_results_text: "没有找到此项!"	
});
```
```
//默认参数。为了减少用户配置项的数量，也为了程序的健壮，默认参数是十分必要的。
$.fn.ActivityOtherTitleEditor.defaults = {
	//最多能添加个数
	maxLength : 10
}
```
### 小结
以上拆解了整个jquery插件结构，其中有一些js和jquery的基础知识大家可以直接上网查到资料，无庸赘述。最后，言语拙漏，欢迎指正。

