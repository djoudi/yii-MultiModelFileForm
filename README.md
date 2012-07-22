## Steps to add a child model inside a parent model's form (with and without a file field):

This example will use Band as the parent model and BandImage as the child model.

### Step 1: Make sure that the child model contains a "weight" attribute which will be used to order the child models

### Step 2: Modify the parent model "create" controller to include the child model and MultiModelForm validate() and save() method calls

Required variables to create:
* $bandImage
* $bandImages
* $validatedImages

If child model contains a file field the following array must be populated:
* $fileAttributes (see required keys below)

```php
<?php
public function actionCreate(){
	// The parent model (Gii creates this automatically)
	$model = new Band;
	// The child model
	$bandImage = new BandImage;
	// An array to hold multiple child models (used in the form view)
	$bandImages = array();
	// An array to hold all validated child models
	$validatedImages = array();
	// An array of arrays that hold upload file information for each model file attribute
	// Keys "attribute" and "destination" must be populated
	$fileAttributes = array(
		array('attribute' => 'image_id', 'destination' => 'media/images/band_images'),
	);
	// If $_POST values are present
	if(isset($_POST['Band'])){
		$model->attributes=$_POST['Band'];
		// First check if the child models validate, then check if the parent model saves (neither can fail validation)
		if(MultiModelForm::validate($projectItem, $validatedItems, $deleteItems, $fileAttributes) && $model->save()){
			// Create an array to hold the foreign key values of the child model
			$masterValues = array('band_id'=>$model->id);
			// Save the child models
			MultiModelForm::save($bandImage, $validatedImages, $deleteImages, $masterValues, $fileAttributes);
			// Do something on success...
			Yii::app()->user->setFlash('band', $model->name . ' created!');
			$this->redirect(array('admin'));                
		}       
	}
	// Send variables to the view
	$this->render('create',array(
		'model'=>$model,
		'bandImage'=>$bandImage,
		'bandImages'=>$bandImages,
		'validatedImages'=>$validatedImages,
	));
}
```

### Step 3: Modify the parent model "create" view to pass the variables to the "_form", which is rendered using a "partialRender"
```php
<?php echo $this->renderPartial('_form', array(
	'model'=>$model,
	'bandImage' => $bandImage,
	'bandImages' => $bandImages,
	'validatedItems' => $validatedItems,
)); ?>
```

### Step 4: Modify the parent model "_form.php" to include the MultiModelForm form

First, create a parent model method that returns a form config object, used to display the child model form (created in Band.php):

```php
<?php
public function getBandImageForm(){
	return array(
		'elements'=>array(
			// Create form elements for each child model attribute
			'image_id'=>array(
				'type'=>'file',
			),
	));
}
```

Second, add the widget call to the parent model "_form.php" file

```php
<?php
$this->widget('ext.multimodelform.MultiModelForm',array(
		'id' => 'project-images',
		'formConfig' => $model->getBandImageForm(),
		'removeText' => 'Remove item',
		'addItemText' => '+ Add Band image',
		'model' => $bandImage,
		'sortAttribute' => 'weight',
		'validatedItems' => $validatedItems, 
		'data' => $bandImages,
	));
?>
```

### Step 5: Modify the parent model "update" controller to include the child model and MultiModelForm validate() and save() calls

Required variables to create:
* $bandImage
* $bandImages
* $validatedImages

If child model contains a file field the following array must be populated:
* $fileAttributes (see required keys below)

```php
<?php
public function actionUpdate($id){
	// The parent model (Gii creates this automatically)
	$model = new Band;
	// The child model
	$bandImage = new BandImage;
	// Get all child models where the foreign key matches the parent model ID
	// If the child model contains a file field, make sure to include the file relation, example: with('file')
	$bandImages = BandImage::model()->with('file')->findAllByAttributes(array('band_id'=>$id), array('order'=>'weight'));
	// An array to hold all validated child models
	$validatedImages = array();
	// An array of arrays that hold upload file information for each model file attribute
	// Keys "attribute" and "destination" must be populated
	$fileAttributes = array(
		array('attribute' => 'image_id', 'destination' => 'media/images/band_images'),
	);
	// If $_POST values are present
	if(isset($_POST['Band'])){
		$model->attributes=$_POST['Band'];
		// Create an array to hold the foreign key values of the child model
		$masterValues = array('band_id'=>$model->id);
		// Check if both the child and parent models validate and save
		if(MultiModelForm::save($bandImage, $validatedImages, $deleteImages, $masterValues, $fileAttributes) && $model->save()){
			// Do something on success...
			Yii::app()->user->setFlash('band', $model->name . ' updated!');
			$this->redirect('/admin');
		}
	}
	// Send variables to the view
	$this->render('update',array(
		'model'=>$model,
		'bandImage'=>$bandImage,
		'bandImages'=>$bandImages,
		'validatedImages'=>$validatedImages,
	));
}
```

### Step 6: Modify the parent model "update" view to pass the variables to the "_form", which is rendered using a "partialRender"
```php
<?php echo $this->renderPartial('_form', array(
	'model'=>$model,
	'bandImage' => $bandImage,
	'bandImages' => $bandImages,
	'validatedItems' => $validatedItems,
)); ?>
```
### Step 7: If the child model contains a file field, be sure to set the child model "rules" to allow the file field to be empty:

```php
<?php
// Sample way to set the file field to allow empty (this can be customized)
array('image_id', 'file', 'types'=>'jpg, jpeg, png, gif', 'allowEmpty' => TRUE),