---
layout: post
title: 404 页面，其实我们可以做的更多
categories:
- 未分类
tags: []
published: true
date: '2013-10-18'
permalink: /2013/404-page-we-can-make-a-difference
---
<p>404 页面，除了显示冷冰冰的 404 Not Found 之外，其实我们可以做得更多。QQ空间就把 404页面和失踪的孩子联系到了一起，现在访问到QQ空间的一个任意不存在的页面（404页面），比如：<a href="http://qzone.qq.com/404" target="_blank">http://qzone.qq.com/404</a>，都能看到一个失踪孩子的信息：
<img alt="404" src="http://lux.ssh01.com/blog/wp-content/uploads/2013/10/4041.png" />

将 404 页面和失踪的孩子联系在一起最早是国外一个网站 <a href="http://notfound.org/" target="_blank">notfound.org</a> 提出的想法，号召大家把 404 页面贡献出来，帮助丢失孩子的父母，现在 QQ空间推了国内版本之后，我相信肯定可以帮助更多丢失孩子的父母，所以我也第一时间在自己的博客上用上。</p>

<p>对于 WP 博客来说，实现这个功能非常简单，只需要将当前主题的的 404.php中添加:</p>

	<?php echo( '<script type="text/javascript" src="http://www.qq.com/404/search_children.js" charset="utf-8"></script>' ); ?>
	#you can also put <script>..</script> directly, without wrapping it in <?php> ... ?>
	




<p>对于我所用的这个模版，最终代码类似这样：</p>

	<?php
	/**
	* The template for displaying 404 pages (Not Found).
	*
	* @package WordPress
	* @subpackage Twenty_Twelve
	* @since Twenty Twelve 1.0
	*/
	 
	get_header(); ?>
	 
	<div id="primary" class="site-content">
	<div id="content" role="main">
	 
	<article id="post-0" class="post error404 no-results not-found">
	<header class="entry-header">
	<h1 class="entry-title"><?php _e( 'This is somewhat embarrassing, isn&rsquo;t it?', 'twentytwelve' ); ?></h1>
	</header>
	 
	<div class="entry-content">
	<p><?php _e( 'It seems we can&rsquo;t find what you&rsquo;re looking for. Perhaps searching can help.', 'twentytwelve' ); ?></p>
	<?php echo( '<script type="text/javascript" src="http://www.qq.com/404/search_children.js" charset="utf-8"></script>' ); ?>
	<?php get_search_form(); ?>
	</div><!-- .entry-content -->
	</article><!-- #post-0 -->
	 
	</div><!-- #content -->
	</div><!-- #primary -->
	 
	<?php get_footer(); ?>
