# Spring Boot  
## 一、起步依赖和自动配置  
### 1. 起步依赖  
当我们想要写一个Spring Boot的工程的时候，根据我们的定位，比如是一个Web应用，那么势必会用到很多库，但是具体我们要用到哪些呢？这恐怕需要一段时间来思考，捋一捋...思考完了呢？一个库可能有很多版本，不同的库的不同的版本，他们能不能互相兼容、正常工作呢？这个...恐怕不是想一想就能知道的问题，这得一个一个坑地去踩才能知道。我们还没开始写Web应用，就已经遇到了这么麻烦的问题，让人想放弃。Spring Boot中的起步依赖就是为了解决这个问题的。  

如果是想要写一个Web应用，与其在配置文件中写入一堆的依赖配置，不如直接声明这是一个Web应用来的方便：  

	compile "org.springframework.boot:spring-boot-starter-web";    
> 可以将起步依赖理解为，自动将你肯能需要的各种依赖加入了项目中，而且保证他们可以正常工作  

当然，起步依赖中引入的依赖有些可能是用不到的，这时就可以在依赖的时候将这个用不到的东西去除：  

	compile ("org.springframework.boot:spring-boot-starter-web"){
		exclude group:'com.fasterxml.jackson.core'
	} 

同样，如果想使用某一模块的更新的版本，可以单独compile一下来覆盖起步依赖中的该项。  

	compile "org.springframework.boot:spring-boot-starter-web";  
	compile("com.fasterxml.jackson.core:jackson-databind:2.4.3")  

如果这个依赖的版本比起步依赖中的要新，那么就会生效；但是如果我们想要用一个旧的版本呢？那就**结合移除和覆盖**：
	
	compile ("org.springframework.boot:spring-boot-starter-web"){
		exclude group:'com.fasterxml.jackson.core'
	}
	compile("com.fasterxml.jackson.core:jackson-databind:2.4.3")

### 2. 自动配置  
自动配置很简单，其主要的表现就是，如果**Classpath里面包含了配置类，那么在使用到的时候回自动生成对应的Bean**；而且还内置了很多的条件注入的注释  



