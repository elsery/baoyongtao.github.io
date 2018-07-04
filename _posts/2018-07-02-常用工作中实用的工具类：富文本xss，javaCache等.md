---
layout: post
title: "常用工作中实用的工具类：富文本xss，java实现带时效Cache等"
date: 2018-07-02
description: "常用工作中实用的工具类"
categories: Java
--- 

###  不定期更新自己认为比较使用的工具类
  

### HashMap 实现带时效的Cache 直接代码了 很简单

```java
package com.example.demo;

import java.util.Map;
import java.util.concurrent.ConcurrentHashMap;

public class HashMapCache {

	private static final String CONDITION_KEY = "condition_key";
	private static final String ACTION_KEY = "action_key";

	/**
	 * 预缓存信息
	 */
	private static final Map<String, Object> CACHE_MAP = new ConcurrentHashMap<String, Object>();
	/**
	 * 每个缓存生效时间12小时
	 */
	public static final long CACHE_HOLD_TIME_12H = 12 * 60 * 60 * 1000L;

	/**
	 * 每个缓存生效时间24小时
	 */
	public static final long CACHE_HOLD_TIME_24H = 24 * 60 * 60 * 1000L;

	/**
	 * 存放一个缓存对象，默认保存时间12小时
	 * 
	 * @param cacheName
	 * @param obj
	 */
	public static void put(String cacheName, Object obj) {
		put(cacheName, obj, CACHE_HOLD_TIME_12H);
	}

	/**
	 * 存放一个缓存对象，保存时间为holdTime
	 * 
	 * @param cacheName
	 * @param obj
	 * @param holdTime
	 */
	public static void put(String cacheName, Object obj, long holdTime) {
		CACHE_MAP.put(cacheName, obj);
		CACHE_MAP.put(cacheName + "_HoldTime", System.currentTimeMillis() + holdTime);// 缓存失效时间
	}

	/**
	 * 取出一个缓存对象
	 * 
	 * @param cacheName
	 * @return
	 */
	public static Object get(String cacheName) {
		if (checkCacheName(cacheName)) {
			return CACHE_MAP.get(cacheName);
		}
		return null;
	}

	/**
	 * 删除所有缓存
	 */
	public static void removeAll() {
		CACHE_MAP.clear();
	}

	/**
	 * 删除某个缓存
	 * 
	 * @param cacheName
	 */
	public static void remove(String cacheName) {
		CACHE_MAP.remove(cacheName);
		CACHE_MAP.remove(cacheName + "_HoldTime");
	}

	/**
	 * 检查缓存对象是否存在， 若不存在，则返回false 若存在，检查其是否已过有效期，如果已经过了则删除该缓存并返回false
	 * 
	 * @param cacheName
	 * @return
	 */
	public static boolean checkCacheName(String cacheName) {
		Long cacheHoldTime = (Long) CACHE_MAP.get(cacheName + "_HoldTime");
		if (cacheHoldTime == null || cacheHoldTime == 0L) {
			return false;
		}
		if (cacheHoldTime < System.currentTimeMillis()) {
			remove(cacheName);
			return false;
		}
		return true;
	}

	public static String getCondition() {
		String object = (String) get(CONDITION_KEY);
		if (object == null) {
			put(CONDITION_KEY, "1111111111111", 5000);
			System.out.println("开始新的缓存查询！");
		}
		return "1111111111111";
	}

	public static void main(String[] args) throws Exception {
		while (true) {
			String condition = getCondition();
			System.out.println(condition);
			Thread.sleep(1000);
		}
	}

}

```


### 对于富文本的xss防范，过滤脚本标签，触发的事件，变换的xss类型，需要依赖

```xml
	<dependency>
		<groupId>oro</groupId>
		<artifactId>oro</artifactId>
		<version>2.0.8</version>
	</dependency>
```
####代码

```java
package tes;

import java.util.regex.Matcher;
import java.util.regex.Pattern;

import org.apache.oro.text.regex.PatternCompiler;
import org.apache.oro.text.regex.Perl5Compiler;
import org.apache.oro.text.regex.Perl5Matcher;
import org.apache.oro.text.regex.Perl5Substitution;
import org.apache.oro.text.regex.Util;

/***
 * 过滤富文本的xss
 * 
 * 
 */
public class SobotXSS {

	static final Pattern SCRIPT_TAG_PATTERN = Pattern.compile("<script[^>]*>.*</script[^>]*>",
			Pattern.CASE_INSENSITIVE);

	static final PatternCompiler pc = new Perl5Compiler();
	static final Perl5Matcher matcher = new Perl5Matcher();

	public static String antiXSS(String content) {
		if (content == null || "".equals(content)) {
			return "";
		}
		String old = content;
		String ret = _antiXSS(content);
		while (!ret.equals(old)) {
			old = ret;
			ret = _antiXSS(ret);
		}
		return ret;
	}

	private static String _antiXSS(String content) {
		try {
			return stripAllowScriptAccess(
					stripProtocol(stripCssExpression(stripAsciiAndHex(stripEvent(stripScriptTag(content))))));
		} catch (Exception e) {
			e.printStackTrace();
			return null;
		}
	}

	private static String stripScriptTag(String content) {
		Matcher m = SCRIPT_TAG_PATTERN.matcher(content);
		content = m.replaceAll("");
		return content;
	}

	private static String stripEvent(String content) throws Exception {
		String[] events = { "onmouseover", "onmouseout", "onmousedown", "onmouseup", "onmousemove", "onclick",
				"ondblclick", "onkeypress", "onkeydown", "onkeyup", "ondragstart", "onerrorupdate", "onhelp",
				"onreadystatechange", "onrowenter", "onrowexit", "onselectstart", "onload", "onunload",
				"onbeforeunload", "onblur", "onerror", "onfocus", "onresize", "onscroll", "oncontextmenu" };
		for (String event : events) {
			org.apache.oro.text.regex.Pattern p = pc.compile("(<[^>]*)(" + event + ")([^>]*>)",
					Perl5Compiler.CASE_INSENSITIVE_MASK);
			if (null != p)
				content = Util.substitute(matcher, p, new Perl5Substitution("$1" + event.substring(2) + "$3"), content,
						Util.SUBSTITUTE_ALL);

		}
		return content;
	}

	private static String stripAsciiAndHex(String content) throws Exception {
		org.apache.oro.text.regex.Pattern p = pc.compile("(<[^>]*)(&#|\\\\00)([^>]*>)",
				Perl5Compiler.CASE_INSENSITIVE_MASK);
		if (null != p)
			content = Util.substitute(matcher, p, new Perl5Substitution("$1$3"), content, Util.SUBSTITUTE_ALL);
		return content;
	}

	private static String stripCssExpression(String content) throws Exception {
		org.apache.oro.text.regex.Pattern p = pc.compile("(<[^>]*style=.*)/\\*.*\\*/([^>]*>)",
				Perl5Compiler.CASE_INSENSITIVE_MASK);
		if (null != p)
			content = Util.substitute(matcher, p, new Perl5Substitution("$1$2"), content, Util.SUBSTITUTE_ALL);

		p = pc.compile("(<[^>]*style=[^>]+)(expression|javascript|vbscript|-moz-binding)([^>]*>)",
				Perl5Compiler.CASE_INSENSITIVE_MASK);
		if (null != p)
			content = Util.substitute(matcher, p, new Perl5Substitution("$1$3"), content, Util.SUBSTITUTE_ALL);

		p = pc.compile("(<style[^>]*>.*)/\\*.*\\*/(.*</style[^>]*>)", Perl5Compiler.CASE_INSENSITIVE_MASK);
		if (null != p)
			content = Util.substitute(matcher, p, new Perl5Substitution("$1$2"), content, Util.SUBSTITUTE_ALL);

		p = pc.compile("(<style[^>]*>[^>]+)(expression|javascript|vbscript|-moz-binding)(.*</style[^>]*>)",
				Perl5Compiler.CASE_INSENSITIVE_MASK);
		if (null != p)
			content = Util.substitute(matcher, p, new Perl5Substitution("$1$3"), content, Util.SUBSTITUTE_ALL);
		return content;
	}

	private static String stripProtocol(String content) throws Exception {
		String[] protocols = { "javascript", "alert", "vbscript", "livescript", "ms-its", "mhtml", "data", "firefoxurl",
				"mocha" };
		for (String protocol : protocols) {
			org.apache.oro.text.regex.Pattern p = pc.compile("(<[^>]*)" + protocol + ":([^>]*>)",
					Perl5Compiler.CASE_INSENSITIVE_MASK);
			if (null != p)
				content = Util.substitute(matcher, p, new Perl5Substitution("$1/$2"), content, Util.SUBSTITUTE_ALL);
		}
		return content;
	}

	private static String stripAllowScriptAccess(String content) throws Exception {
		org.apache.oro.text.regex.Pattern p = pc.compile("(<[^>]*)AllowScriptAccess([^>]*>)",
				Perl5Compiler.CASE_INSENSITIVE_MASK);
		if (null != p)
			content = Util.substitute(matcher, p, new Perl5Substitution("$1Allow_Script_Access$2"), content,
					Util.SUBSTITUTE_ALL);
		return content;
	}
}

```