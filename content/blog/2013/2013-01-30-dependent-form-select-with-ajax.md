---
date: "2013-01-30"
draft: false
title: "Dependent form select with AJAX"
tags: ["posobota"]
type: "blog"
slug: "dependent-form-select-with-ajax"
author: "Martin Zlámal"
---

This is case you want to create form select, which values depend on another select value. Simple use case: select country, than select city from the country, than select street from the city.

Requires **jQuery 1.8.2** and [nette.ajax.js](https://addons.nette.org/cs/nette-ajax-js) addon, built on **Nette 2.0.10** sandbox.

## Single dependent select

Logic is very simle: when first select is changed a `handle*` method is called by AJAX in background (in this case `handleFirstSelect($value)`). This method specifies values for second select, loads select input with them and invalidates snippet.

If you don't know, what *snippet* or *Nette Ajax* is, I recommend you to check [Quickstart](doc:ajax).


**HomepagePresenter.php**

```php
use Nette\Application\UI\Form;

class HomepagePresenter extends BasePresenter
{

	public function renderDefault()
	{
		// required to enable form access in snippets
		$this->template->_form = $this['selectForm'];
	}

	/**
	 * Load values to second select
	 * @param int
	 */
	public function handleFirstChange($value)
	{
		if ($value) {
			$secondItems = array(
				1 => 'First option ' . $value . ' - second option 1',
				2 => 'First option ' . $value . ' - second option 2'
			);

			$this['selectForm']['second']->setPrompt('Select')
				->setItems($secondItems);

		} else {
			$this['selectForm']['second']->setPrompt('Select from first')
				->setItems(array());
		}

		$this->invalidateControl('secondSnippet');
	}


	/********************** form **********************/


	protected function createComponentSelectForm()
	{
		$firstItems = array(
			1 => 'First option 1',
			2 => 'First option 2'
		);

		$form = new Form;
		$form->addSelect('first', 'First select:', $firstItems)
			->setPrompt('Select');
		$form->addSelect('second', 'Second select:')
			->setPrompt('Select from first');
		$form->addSubmit('send', 'Submit');

		$form->onSuccess[] = $this->processSelectForm;

		return $form;
	}


	/**
	 * @param form
	 */
	public function processSelectForm(Form $form)
	{
		// $form->getValues() ignores invalidated input's values
		$values = $form->getHttpData();
		unset($values['send']);
		dump($values);

		// ...
	}

}
```

**Homepage/default.latte**

```html
{define #content}

{form selectForm}
	{label first /} {input first}
	{snippet secondSnippet}
		{label second /} {input second}
	{/snippet}
	{input send}
{/form}

<script>
{include #jsCallback, input => first, link => firstChange}
</script>

{/define}


{define #jsCallback}

$('#{$control["selectForm"][$input]->htmlId}').on('change', function() {
	$.nette.ajax({
		type: 'GET',
		url: '{link {$link}!}',
		data: {
			'value': $(this).val(),
		}
	});
});

{/define}
```



Multiple dependent selects
===

**HomepagePresenter.php**

```php
use Nette\Application\UI\Form;

class HomepagePresenter extends BasePresenter
{

	public function renderDefault()
	{
		$this->template->_form = $this['selectForm'];
	}


	/**
	 * Load values to second select
	 * @param int
	 */
	public function handleFirstChange($value)
	{
		if ($value) {
			$secondItems = array(
				1 => 'First option ' . $value . ' - second option 1',
				2 => 'First option ' . $value . ' - second option 2'
			);

			$this['selectForm']['second']->setPrompt('Select')
				->setItems($secondItems);

			$this['selectForm']['third']->setPrompt('Select from second');

		} else {
			$this['selectForm']['second']->setPrompt('Select from first')
				->setItems(array());

			$this['selectForm']['third']->setPrompt('Select from first')
				->setItems(array());
		}

		$this->invalidateControl('secondSnippet');
	}


	/**
	 * Load values to third select
	 * @param int
	 */
	public function handleSecondChange($value)
	{
		if ($value) {
			$thirdItems = array(
				1 => 'Second option ' . $value . ' - third option 1',
				2 => 'Second option ' . $value . ' - third option 2'
			);

			$this['selectForm']['third']->setPrompt('Select')
				->setItems($thirdItems);

		} else {
			$this['selectForm']['third']->setPrompt('Select from second')
				->setItems(array());
		}

		$this->invalidateControl('thirdSnippet');
	}


	/********************** form **********************/


	protected function createComponentSelectForm()
	{
		$firstItems = array(
			1 => 'First option 1',
			2 => 'First option 2'
		);

		$form = new Form;
		$form->addSelect('first', 'First select:', $firstItems)
			->setPrompt('Select');
		$form->addSelect('second', 'Second select:')
			->setPrompt('Select from first');
		$form->addSelect('third', 'Third select:')
			->setPrompt('Select from first');
		$form->addSubmit('send', 'Submit');

		$form->onSuccess[] = $this->processSelectForm;

		return $form;
	}


	/**
	 * @param form
	 */
	public function processSelectForm(Form $form)
	{
		// same as above
	}

}
```


**Homepage/default.latte**

To keep ajax alive for all snippets, we have to place it into last changing one (in this case into `{snippet secondSnippet}`).

```html
{define #content}

{form selectForm}
	{label first /} {input first}
	{snippet secondSnippet}
		{label second /} {input second}

		{snippet thirdSnippet}
			{label third /} {input third}
		{/snippet}

		{include #js}
	{/snippet}
	{input send}
{/form}

{/define}


{define #js}

<script type="text/javascript">
{include #jsCallback, input => first, link => firstChange}
{include #jsCallback, input => second, link => secondChange}
</script>

{/define}


{define #jsCallback}
{* same as above *}
{/define}
```
