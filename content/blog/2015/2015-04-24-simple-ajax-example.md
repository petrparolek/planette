---
date: "2015-04-24"
draft: false
title: "Simple ajax example"
tags: ["ajax"]
type: "blog"
slug: "simple-ajax-example"
author: "Tomáš Votruba"
---

This tutorial is created using **Nette 2.0.10** Sandbox.


## Setup

First, we download [Nette](www:download) and use content of `sandbox` folder as base for our tutorial.

Than we need to link [Nette Ajax addon](http://addons.nette.org/cs/nette-ajax-js). Lets download current version of [nette.ajax.js](https://github.com/vojtech-dobes/nette.ajax.js). Place it to `www/js` folder.


**Templates/@layout.latte**

```latte
{block scripts}
<script src="{$basePath}/js/jquery.js"></script>
<script src="{$basePath}/js/netteForms.js"></script>
<script src="{$basePath}/js/nette.ajax.js"></script> {* Nette Ajax depens on jQuery *}
<script src="{$basePath}/js/main.js"></script>
{/block}
```

Than we initiate Nette Ajax in `main.js`:

```js
$(function () {
    $.nette.init();
});
```

Simple `$.nette.init()` enables link and form ajaxization via class `.ajax`. To get more info see [usage](https://github.com/vojtech-dobes/nette.ajax.js#usage) on addon page.


## In presenter

This is the most simple use of Nette Ajax. Here we go!

**HomepagePresenter.php**

```php
class HomepagePresenter extends BasePresenter
{
	/** @var string */
	private $anyVariable;


	public function handleChangeVariable()
	{
		$this->anyVariable = 'changed value via ajax';
		if ($this->isAjax()) {
			$this->invalidateControl('ajaxChange');
		}
	}


	public function renderDefault()
	{
		if ($this->anyVariable === NULL) {
			$this->anyVariable = 'default value';
		}
		$this->template->anyVariable = $this->anyVariable;
	}

}

```


To change value in template we need to use special property `$anyVariable`. If we would use `$this->template->anyVariable = '...'` both in `handle*` and `render*`, only `render*` would matter, see [life cycle of presenter](doc:presenters#toc-life-cycle-of-presenter).

**Homepage/default.latte**

```latte
<div id="content">
	{snippet ajaxChange}
		{$anyVariable}
	{/snippet}

	<a n:href="changeVariable!" class="ajax">Change variable!</a>
</div>
```

Now just go and click on **Change variable!** link. *default value* should change to "changed value via ajax".


## In component

If you want to use ajax in component, you can check [ajax in documentation](doc:ajax).

```comment
Fix quickstart:ajax to en version when ready
```
