# Steps to add model inside another model's form (with and without a file field):

This example will use Band as the parent model and BandImage as the child model.

## Step 1: Modify the parent model "create" controller to include the child model and MultiModelForm validate() and save() method calls

Required variables to create:
* $bandImage
* $bandImages
* $validatedImages



```php
<?php
public function actionCreate(){
	$model = new Band;
	$bandImage = new BandImage;
	$bandImages = array();
	$validatedImages = array();
	$fileAttributes = array(
		array('attribute' => 'image_id', 'destination' => 'media/images/band_images'),
	);
	
	if(isset($_POST['Band'])){
		$model->attributes=$_POST['Band'];
			
		// check if the model validates
		if($model->save()){
			$masterValues = array('band_id'=>$model->id);
			MultiModelForm::save($bandImage, $validatedImages, $deleteImages, $masterValues, $fileAttributes);
			Yii::app()->user->setFlash('band', $model->name . ' created!');
			$this->redirect(array('admin'));				
		}		
	}
  
	$this->render('create',array(
		'model'=>$model,
		'bandImage'=>$bandImage,
		'bandImages'=>$bandImages,
		'validatedImages'=>$validatedImages,
	));
}
```

