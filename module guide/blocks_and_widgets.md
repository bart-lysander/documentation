# Blocks and widgets

## What's the difference?

Blocks and widgets are the visible parts of your websites. The blocks, you'll find in the actions folder, the widgets you'll find in the widgets folder. Their operation is exactly the same with two differences:

* There can only be one “block” on a page.
* Because of this, each block has an URL (that can be fetched using the function getUrlForBlock)

This block can be a module, with every action included, or just one action of a module.

As long as the selected template allows it, there can be as many widgets as you want on a page. The structure of a widget is (most of the time) less complicated than a block because they are used merely for displaying data.

## Structure

When you check out other existing Fork modules, you will see that most actions use the same structure, using the same method names. Beneath you'll find the complete code of the detail-action of our mini blog.

```
class FrontendMiniBlogDetail extends FrontendBaseBlock
{
```

Again the classname needs the be exact ApplicationModuleAction, in our case FrontendMiniBlogDetail. Because our action is a Block, it extends FrontendBaseBlock. This class takes care of everything concerning URL-handling, breadcrumbs, ...

```
/** 
 * The blogpost 
 * 
 * @vararray 
 */ 
private $record; 
```

Then we define our (private) variables. In our case we'll use an array to save the record with the article we will be viewing.

```
public function execute()
{
 // call the parent 
 parent::execute();

 // hide contenTitle, in the template the title is wrapped with an inverse-option
 $this->tpl->assign('hideContentTitle', true); 

 // load template 
 $this->loadTemplate(); 

 // load the data 
 $this->getData(); 

 // parse 
 $this->parse(); 
}
```

The execute function is always present and is called by Fork CMS when opening any action. As you can see, the execute method of FrontendBaseBlock is called too. This makes sure that the js-files and css-files are autoloaded.

The line starting with "$this->tpl->assign(" ... assigns a variable to the template we'll be using to display the action.

loadTemplate (also defined in FrontendBaseBlock) loads the template file in which we parse the data we'll be loading in our self defined method getData.

```
 /*
 * Load the data
 *
 * @return void
 */
private function getData()
{
 // if no parameter was passed we redirect to the 404-page
 if($this->URL->getParameter(1) === null) $this->redirect(FrontendNavigation::getURL(404));

 // get the record, or at least try it
 $this->record = FrontendMiniBlogModel::get($this->URL->getParameter(1));

 // if the record is empty it is an invalid one, so redirect to the 404-page
 if(empty($this->record)) $this->redirect(FrontendNavigation::getURL(404));

 // add some extra info to the record
 $this->record['full_url'] = FrontendNavigation::getURLForBlock('mini_blog' , 'detail') . '/' . $this->record['url'];
 $this->record['tags'] = FrontendTagsModel::getForItem('mini_blog' , $this->record['id']);
} 
```

The getData function first checks if the item given in the URL exists and adds some extra data which will be used in the template, or redirects to a 404-page if it doesn't. (The 404 page is installed by default when installing Fork).
If an article was found, the data we fetched is parsed into the template-file.

```
private function parse()
{
 $this->breadcrumb->addElement($this->record['title']);

 $this->header->setPageTitle($this->record['title']);
 $this->header->setMetaDescription($this->record['meta_description'] , ($this->record['meta_description_overwrite'] == 'Y'));
 $this->header->setMetaKeywords($this->record['meta_keywords'] , ($this->record['meta_keywords_overwrite'] == 'Y'));

 $this->tpl->assign('item', $this->record);
 $this->tpl->assign('navigation' , FrontendMiniBlogModel::getNavigation($this->record['id']));
 }
}
```

As you can see, it's fairly easy to add an item to Fork's breadcrumb object and to add the meta-data to the <head> of the page. We discuss both objects later on.