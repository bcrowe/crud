# Filling Related Models select boxes

If you are used to bake or CakePHP scaffolding you might want to have some control over the data it is sent to the view for
filling select boxes for associated models. Crud component can be configured to return the list of record for all related models
or just those you want to in a per-action basis

By default all related model lists for main Crud component model instance will be fetched, but only for `add`, `edit` and corresponding
admin actions. For instance if your `Post` model in associated to `Tag` and `Author`, then for the aforementioned actions you will have
in your view the `authors` and `tags` variable containing the result of calling find('list') on each model.

Should you need more fine grain control over the lists fetched, you can configure statically or use dynamic methods:

```php
<?php
class DemoController extends AppController {
   /**
	* List of global controller components
	*
	* @cakephp
	* @var array
	*/
	public $components = array(
		// Enable CRUD actions
		'Crud.Crud' => array(
			'actions' => array('index', 'add', 'edit', 'view', 'delete'),
			'relatedList' => array(
				'add' => array('Author'), //Only set $authors variable in the view for action add and admin_add
				'edit' => array('Tag', 'Cms.Page'), //Set $tags and $pages variable. Page model from plugin Cms will be used
				// As admin_edit is not listed here it will use defaults from edit action
			)
		)
	);
}
?>
```

You can also configure default to not repeat yourself too much:

```php
<?php
class DemoController extends AppController {
   /**
	* List of global controller components
	*
	* @cakephp
	* @var array
	*/
	public $components = array(
		// Enable CRUD actions
		'Crud.Crud' => array(
			'actions' => array('index', 'add', 'edit', 'view', 'delete'),
			'relatedList' => array(
				'default' => array('Author'),
				'add' => true, // add action is enabled and will fetch Author by default
				'admin_change' => true, // admin_change action is enabled and will fetch Author by default
				'edit' => array('Tag'), //edit action is enabled and will only fetch Tags
				'admin_edit' => false // admin_edit action is disabled, no related models will be fetched
			)
		)
	);
}
?>
```

If configuring statically is not your thing, or you want to dynamically fetch related models based on some conditions, then you can
call `mapRelatedList` and `enableRelatedList` function in CrudComponent:


```php
<?php
class DemoController extends AppController {
	public function beforeFilter() {
		parent::beforeFilter();
		$this->Crud->enableRelatedList(array('index', 'delete'));
		$this->mapRelatedList(array('Author', 'Cms.Page'), 'default'); // By default all enabled actions should fetch Author and Page
	}


	public function delete() {
		$this->mapRelatedList(array('Author'), 'default'); // Only fetch authors list
		$this->Crud->executeAction('delete');
	}

}
?>
```

## Related models' list events

If for any reason you need to alter the query or final results generated by fetching related models lists, you can use `Crud.beforeListRelated` and
`Crud.afterListRelated` events to inject your own logic.

`Crud.beforeListRelated` wil receive the following parameters in the event subject, which can be altered on the fly before any result is fetched

	* query: An array with options for find('list')
	* model: Model instance, the model to be used for fiding the list or records


`Crud.afterListRelated` wil receive the following parameters in the event subject, which can be altered on the fly after results were fetched

	* items: result from calling find('list')
	* viewVar: Variable name to be set on the view with items as value
	* model: Model instance, the model to be used for fiding the list or records


## Example

```php
<?php
class DemoController extends AppController {
	//...

	public function beforeFilter() {
		parent::beforeFilter();

		//Authors list should only have the 3 most recen items
		$this->Crud->on('beforeListRelated', function($event) {
			if ($event->subject->model instanceof Author) {
				$event->subject->query['limit'] = 3;
				$event->subject->query['order'] = array('Author.created' => 'DESC');
			}
		});

		$this->Crud->on('afterListRelated', function($event) {
			if ($event->subject->model instanceof Tag) {
				$event->subject->items += array(0 => 'N/A');
				$event->subject->viewVar = 'labels';
			}
		});
	}

}
?>
```