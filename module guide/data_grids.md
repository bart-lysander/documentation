# Data grids

Probably for every type of backend page you can think of, you'll want to have an overview of the items. That's where the data grids come in.

## Fetch data

The first thing you have to define is an SQL statement you'll use to fetch your items.
Typically you'll do this is the backend model.php of your module:

```
class BackendMiniBlogModel
{
	const QRY_DATAGRID_BROWSE = 
		'SELECT i.id, i.title, UNIX_TIMESTAMP(i.created) AS created, i.user_id 
		 FROM mini_blog AS i WHERE i.publish = ? AND i.language = ?';
```
		 
As you can see it's as always a nice piece of parametrized SQL. Mind that there is no ORDER BY clause. This will be provided by the datagrid.

## Define

In the index-action, we define a variable to save our data grid

```
class BackendMiniBlogIndex extends BackendBaseActionIndex
{
	/**
	 * datagrid with published items
	 *
	 * @var	SpoonDataGrid
	 */
	private $dgPublished;


	/**
	 * datagrid with unpublished items
	 *
	 * @var	SpoonDataGrid
	 */
	private $dgNotYetPublished;
```

As you can see, in our example we'll be using two datagrids.

As with every action we have an execute function. It's structure however is different from the previous actions we encountered. We don't need methods to load or validate forms, but have a function that generates the data grid. Depending on you module, this function will almost always be different. Sometimes you'll even need a couple of functions to generated a couple of data grids. In our case we can re-use the function by supplying an argument.

```
public function execute()
{	
	parent::execute();

	$this->dgPublished = $this->loadDataGrid('Y');

	$this->dgNotYetPublished = $this->loadDataGrid('N');

	$this->parse();

	$this->display();
}
```

As you can see the loadDataGrid function uses the SQL-statement we defined in the model and passes two arguments.

```
private function loadDataGrid($published)
{
	$dg = new BackendDataGridDB(BackendMiniBlogModel::QRY_DATAGRID_BROWSE,
					array($published, BL::getWorkingLanguage()));
```
					
Standardly all the columns that are returned by the SQL are show in the data grid with the column names used as a label. Should you want to use another label, you call the method "setHeaderLabels" supplied with an array that defines which column names should be replace by which "word". As you can see, our "word" is another label.

```
	$dg->setHeaderLabels(array('user_id' => ucfirst(BL::lbl('Author'))));
```

If it should be possible for the user to sort the datagrid by clicking the column names you have to provide which columns.

```
	$dg->setSortingColumns(array('created', 'title', 'user_id'), 'created');
	$dg->setSortParameter('desc');
```

The last argument of the setSortingColumns functions is the default column, which will be sorted in the default order defined by setSortParameter.
You can make a field in clickable per column. In our case, we want to go to the edit page of the clicked article when the users clicks the title column

```
$dg->setColumnURL('title', BackendModel::createURLForAction('edit') . '&amp;id=[id]');
```

The same way you can add modifiers to variables in templates, you can apply functions to the data grids. In this example we transfer our datetime value to a nice readable date and instead of the user-id we print the username.

```
$dg->setColumnFunction(array('BackendDatagridFunctions', 'getLongDate'),
				array('[created]'), 'created', true);

$dg->setColumnFunction(array('BackendDatagridFunctions', 'getUser'),
				array('[user_id]'), 'user_id', true);
```

You can add extra columns as well. There are different types of columns you can add. Here we add an edit and delete column. By adding it this way, they automagically get a different layout.

```
	$dg->addColumn('edit', null, BL::lbl('Edit'), BackendModel::createURLForAction('edit') .
 				'&amp;id=[id]', BL::lbl('Edit'));

	$dg->addColumn('delete', null, BL::lbl('Delete'), 		
		BackendModel::createURLForAction('delete') . '&amp;id=[id]', BL::lbl('Delete'));
```

Perhaps you remember from a couple of pages ago we added the GET-variables `highlight=row-' . $item['id']` to our url when directing after editing.

In the next line we add an id-attribute to the rows of the table. When the datagrid is parsed later on, Fork CMS will check these id's, in combination with the given highlight- GET-variable an add an extra css-class to the row when there is a match.

```
	$dg->setRowAttributes(array('id' => 'row-[id]'));

	return $dg;
}
```