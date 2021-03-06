---
title: Spring中bean的作用域
date: 2019-01-16 14:40:00
author: CarleViets
img: 
top: # If top value is true, it will be the homepage recommendation post
# If you want to set the reading verification password for the post, 
# you can set the password value, which must be encrypted with SHA256 to prevent others from seeing it.
password: 
# Does this post open mathjax, Need to be activated in the theme's _config.yml.
mathjax: false
categories: HelloWorld
tags:
  - spring
---

# 


## 一、bean的四种作用域

Spring定义了多种作用域，可以基于这些作用域创建bean，包括:

	单例（Singleton）：在整个应用中，只创建bean的一个实例。
	原型（Prototype）：每次注入或者通过Spring应用上下文获取的时候，都会创建一个新的bean实例。
	会话（Session）：在Web应用中，为每个会话创建一个bean实例。
	请求（Rquest）：在Web应用中，为每个请求创建一个bean实例 

在默认情况下， Spring应用上下文中所有bean都是作为以单例（singleton）的形式创建的。也就是说，不管给定的一个bean被注入到其他bean多少次，每次所注入的都是同一个实例。在大多数情况下，单例bean是很理想的方案。但是在某些情况下，你所使用的类是易变的（mutable），它们会保持一些状态，因此重用是不安全的。在这种情况下，将class声明为单例的bean就不是什么好主意了。

## 二、使用其他作用域的bean
要声明bean为其他作用域，可以使用@Scope注解，以下是用过@Scope注解和@Component注解一起使用，声明了一个原型作用域的bean：
	
	@Component
	@Scope(ConfigurableBeanFactory.SCOPE_PROTOTYPE)
	public class Notepad{...}

这里，使用ConfigurableBeanFactory类的`SCOPE_PROTOTYPE`常量设置了原型作用域。你当然也可以使用@Scope("prototype")，但是使用`SCOPE_PROTOTYPE`常量更加安全并且不易出错。

同样，@Scope注解也可以和@Bean注解一起使用:

	@Bean
	@Scope(ConfigurableBeanFactory.SCOPE_PROTOTYPE)
	public Notepad notepad(){
		return new Notepad();
	}

同样，可以使用scope属性通过XML的方式配置其他作用域的bean:

	<bean id="notepad" class="com.carleviets.Notepad" scope="prototype"/>


## 三、使用会话和请求作用域

在典型的电子商务应用中，可能会有一个bean代表用户的购物车。如果购物车是单例的话，那么将会导致所有的用户都会向同一个购物车中添加商品。另一方面，如果购物车是原型作用域的，那么在应用中某一个地方往购物车中添加商品，在应用的另外一个地方可能就不可用了。就购物车bean来说，会话作用域是最为合适的，因为它与给定的用户关联性最大。
	
	@Component
	@Scope(value=WebApplicationContext.SCOPE_SESSION, proxyMode=ScopedProxyMode.INTERFACES)
	public class ShoppingCart{...}

注意,@Scope同时还有一个proxyMode属性，它被设置成了ScopedProxyMode.INTERFACES。这个属性解决了将会话或请求作用域的bean注入到单例bean中所遇到的问题。例如，如下的StoreService bean表示在线商店提供的服务:
	
	@Component
	public class StoreService{
		private ShoppingCart shoppingCart;
		
		@Autowired
		public void setShoppingCart(ShoppingCart shoppingCart){
			this.shoppingCart=shoppingCart;
		}
		...
	}

因为StoreService是一个单例的bean，会在Spring应用上下文加载的时候创建。当它创建的时候， Spring会试图将ShoppingCart bean注入到setShoppingCart()方法中。但是ShoppingCart bean是会话作用域的，此时并不存在。直到某个用户进入系统，创建了会话之后，才会出现ShoppingCart实例。

如果使用了proxyMode=ScopedProxyMode.INTERFACES的话，Spring并不会将实际的ShoppingCart bean注入到StoreService中，而是会注入一个到ShoppingCart bean的代理，如图所示。这个代理会暴露与ShoppingCart相同的方法，所以StoreService会认为它就是一个购物车。但是，当StoreService调用ShoppingCart的方法时，代理会对其进行懒解析并将调用委托给会话作用域内真正的ShoppingCart bean。
	
![avatar](http://wx1.sinaimg.cn/mw690/0060lm7Tly1fz6igknkjlj30r00dy78f.jpg)

这是使用的是Spring基于接口的代理，但如果ShoppingCart是一个具体的类的话，Spring就没有办法创建基于接口的代理了。此时，它必须使用CGLib来生成基于类的代理。所以，如果bean类型是具体类的话，我们必须要将proxyMode属性设置为ScopedProxyMode.TARGET_CLASS，以此来表明要以生成目标类扩展的方式创建代理。

尽管这里主要关注了会话作用域，但是请求作用域的bean会面临相同的装配问题。因此，请求作用域的bean应该也以作用域代理的方式进行注入。




	
