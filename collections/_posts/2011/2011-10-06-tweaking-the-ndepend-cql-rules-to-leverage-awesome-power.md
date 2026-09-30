---
title: Tweaking NDepend CQL rules
date:  2011-10-06 12:00:00 +0100
tags:  sponsored
---

After learning how to automate and schedule NDepend to run for several solutions at once and starting to use NDepend more regularly, the power of CQL has grown on me.

For instance, one problem that I have wrestled is when code contains static fields for fields that should not be static, which in a web context is a big no-no.

Prior to CQL, I used to search for `static` in the entire solution, then go through the search result. It was exhausting and a non-viable approach.

As I dug into the CQL rules, to get a better understanding of the NDepend analysis tools, I noticed the following standard CQL rule:

	// <Name>Static fields should be prefixed with a 's_'</Name>
	WARN IF Count > 0 IN SELECT FIELDS WHERE 
	 !NameLike "^s_" AND 
	 IsStatic AND 
	 !IsLiteral AND 
	 !IsGeneratedByCompiler AND 
	 !IsSpecialName AND 
	 !IsEventDelegateObject 

	// This naming convention provokes debate.
	// Don't hesitate to customize the regex of 
	// NameLike to your preference.

This rule is just awesome! I just had to edit the CQL to

	// <Name>Static fields should not exist...mostly</Name>
	WARN IF Count > 0 IN SELECT FIELDS WHERE 
	 IsStatic AND 
	 !IsLiteral AND 
	 !IsGeneratedByCompiler AND 
	 !IsSpecialName AND 
	 !IsEventDelegateObject 

	// This naming convention provokes debate.
	// Don't hesitate to customize the regex of 
	// NameLike to your preference.

and voilá, NDepend will now automatically find all static fields in the solution and ignore the default naming conventions.

Since this got me going, I also went ahead to modify the following rule

	// <Name>Instance fields should be prefixed with a 'm_'</Name>
	WARN IF Count > 0 IN SELECT FIELDS WHERE 
	 !NameLike "^m_" AND 
	 !IsStatic AND 
	 !IsLiteral AND 
	 !IsGeneratedByCompiler AND 
	 !IsSpecialName AND 
	 !IsEventDelegateObject 

	// This naming convention provokes debate.
	// Don't hesitate to customize the regex of 
	// NameLike to your preference.

to instead require that fields are camel cased:

	// <Name>Instance fields should be camelCase</Name>
	WARN IF Count > 0 IN SELECT FIELDS WHERE 
	 !NameLike "^[a-z]" AND 
	 !IsLiteral AND 
	 !IsGeneratedByCompiler AND 
	 !IsSpecialName AND 
	 !IsEventDelegateObject

These two small changes to the original setup proved to be an insanely helpful time-saver.

Another great thing is that if you edit the queries in `VisualNDepend`, you  get an immediate, visual feedback to how the rule applies to the entire solution.