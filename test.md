# Capture, Crop and Save Image
In this tutorial, we will learn how to capture, crop and save images in android studio. So, let’s get started. We used some external libraries including Andriod Image Cropper and Piccasso. Step by Step Process is as follows:
1.  Select File -> New -> New Project to create a new android studio project. Scroll & Select Empty Activity and click Next.  

3.  Type the project name according to your need. Also, select the minimum SDK so most android users can use your app. And finally, click Finish. 

4. Add the following lines in build.gradle(:app), basically we are adding dependencies to use external libraries in our code.
 implementation 'com.squareup.picasso:picasso:2.5.2'
 api 'com.theartofdev.edmodo:android-image-cropper:2.8.+'






5. To access the android image cropper dependency , we need to add jcenter() in Settings.gradle. After the changes, we need to click “Sync Project” to Sync the project with the latest gradle configuration.

```
dependencyResolutionManagement {
   repositoriesMode.set(RepositoriesMode.FAIL_ON_PROJECT_REPOS)
   repositories {
   	google()
   	mavenCentral()
   	jcenter()
   }
}
```

6.  AndroidManifest.xml has three different sections, first one is adding permissions
```
<uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE"/>
<uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE"/>
<uses-permission android:name="android.permission.CAMERA" />
```


Second is to use Crop image activity from external sources, we need to declare it as an activity in Manifest.
```
<activity android:name="com.theartofdev.edmodo.cropper.CropImageActivity"
  android:theme="@style/Base.Theme.AppCompat"/>
```

Third is to use Camera intent, we need to add following lines in manifest.
```
<queries>
   <!-- Camera -->
   <intent>
       <action android:name="android.media.action.IMAGE_CAPTURE" />
   </intent>
</queries>
```

7. Activity_main.xml is the design section of our app, we used three buttons and only one imageview. It is a  sample code for activity_main.xml (Of course, you can design better). 
[N.B: You can use “Drag and Drop” to design the page instead of writing the xml codes]
```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout
  xmlns:android="http://schemas.android.com/apk/res/android"
  xmlns:tools="http://schemas.android.com/tools"
  android:layout_width="match_parent"
  android:layout_height="match_parent"
  android:layout_marginBottom="100dp"
  android:gravity="center"
  android:orientation="vertical"
  tools:context=".MainActivity">
 
  <!--Here the selected cropped image will be shown-->
  <ImageView
      android:id="@+id/set_profile_image"
      android:layout_width="300dp"
      android:layout_height="300dp"
      android:layout_alignParentTop="true"
      android:layout_centerHorizontal="true"
      android:layout_marginTop="40dp"
      android:src="@drawable/ic_launcher_background" />
 
  <!--Here we are clicking on this text to select
      an image from camera or gallery-->
  <Button
      android:id="@+id/capture"
      android:layout_width="wrap_content"
      android:layout_height="wrap_content"
      android:text="Capture" />
  <Button
      android:id="@+id/save"
      android:layout_width="wrap_content"
      android:layout_height="wrap_content"
      android:text="Save" />
  <Button
      android:id="@+id/crop"
      android:layout_width="wrap_content"
      android:layout_height="wrap_content"
      android:text="Crop" />
</LinearLayout>
```




8. Let’s import these necessary dependencies for MainActivity.	

```

import android.Manifest;
import android.app.AlertDialog;
import android.content.ContentResolver;
import android.content.ContentValues;
import android.content.Context;
import android.content.Intent;
import android.content.pm.PackageManager;
import android.graphics.Bitmap;
import android.graphics.drawable.BitmapDrawable;
import android.net.Uri;
import android.os.Build;
import android.os.Bundle;
import android.provider.MediaStore;
import android.view.View;
import android.widget.Button;
import android.widget.ImageView;
import android.widget.Toast;
import androidx.annotation.NonNull;
import androidx.annotation.Nullable;
import androidx.appcompat.app.AppCompatActivity;
import androidx.core.app.ActivityCompat;
import androidx.core.content.ContextCompat;
import com.squareup.picasso.Picasso;
import com.theartofdev.edmodo.cropper.CropImage;
import java.io.ByteArrayOutputStream;
import java.io.OutputStream;
import java.util.Objects; 
```



9. Variables needed inside MainActivity class are as follows:  
```
ImageView targetframe =null;
private static final int CAMERA_REQUEST = 100;
private static final int STORAGE_REQUEST = 200;
String[] cameraPermission;
String[] storagePermission;
Button capture, crop, save;
```


10. in MainActivity.java while creating the main activity, we linked a defined variable with our xml elements, and added on click listener to add functionalities.
```
protected void onCreate(Bundle savedInstanceState) {
       super.onCreate(savedInstanceState);
       setContentView(R.layout.activity_main);
 
       // Here we are initialising
       // the text and image View
       targetframe = findViewById(R.id.set_profile_image);
       capture = findViewById(R.id.capture);
       crop = findViewById(R.id.crop);
       save = findViewById(R.id.save);
 
       // allowing permissions of gallery and camera
       cameraPermission = new String[]{Manifest.permission.CAMERA, Manifest.permission.WRITE_EXTERNAL_STORAGE};
       storagePermission = new String[]{Manifest.permission.WRITE_EXTERNAL_STORAGE};
       if (!checkCameraPermission()) {
           requestCameraPermission();
       }
 
       // After clicking on text we will have
       // to choose whether to
       // select image from camera and gallery
       crop.setOnClickListener(new View.OnClickListener() {
           @Override
           public void onClick(View view) {
               showImagePicDialog();
           }
       });
       capture.setOnClickListener(new View.OnClickListener() {
           @Override
           public void onClick(View view) {
               if (!checkCameraPermission()) {
                   requestCameraPermission();
               }
               Intent camera_intent = new Intent(MediaStore.ACTION_IMAGE_CAPTURE);
               // Start the activity with camera_intent, and request pic id
               startActivityForResult(camera_intent, 100);
           }
       });
       ActivityCompat.requestPermissions(MainActivity.this, new String[]{Manifest.permission.WRITE_EXTERNAL_STORAGE}, 1);
       ActivityCompat.requestPermissions(MainActivity.this,new String[]{Manifest.permission.WRITE_EXTERNAL_STORAGE},1);
       save.setOnClickListener(new View.OnClickListener() {
 
 
 
           @Override
           public void onClick(View view) {
               if(targetframe !=null) imagesavetomyphonegallery();
 
           }
       });
 
   }
```




11. To give to option to crop from, gallery and camera,alert dialogue introduced in MainActivity.java 
```
   private void showImagePicDialog() {
       String options[] = {"Camera", "Gallery"};
       AlertDialog.Builder builder = new AlertDialog.Builder(this);
       builder.setTitle("Pick Image From");
       builder.setItems(options, (dialog, which) -> {
           if (which == 0) {
               if (!checkCameraPermission()) {
                   requestCameraPermission();
               } else {
                   pickFromCamera();
               }
           } else if (which == 1) {
               if (!checkStoragePermission()) {
                   requestStoragePermission();
               } else {
                   pickFromGallery();
               }
           }
       });
       builder.create().show();
   } 

```






12. Permission is a crucial thing in Android Development, some self explanatory method added to MainActivity to do the task without any error in in MainActivity.java 
```
 
   // checking storage permissions
   private Boolean checkStoragePermission() {
       boolean result = ContextCompat.checkSelfPermission(this, Manifest.permission.WRITE_EXTERNAL_STORAGE) == (PackageManager.PERMISSION_GRANTED);
       return result;
   }
 
   // Requesting gallery permission
   private void requestStoragePermission() {
       if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.M) {
           requestPermissions(storagePermission, STORAGE_REQUEST);
       }
   }
 
   // checking camera permissions
   private Boolean checkCameraPermission() {
       boolean result = ContextCompat.checkSelfPermission(this, Manifest.permission.CAMERA) == (PackageManager.PERMISSION_GRANTED);
       boolean result1 = ContextCompat.checkSelfPermission(this, Manifest.permission.WRITE_EXTERNAL_STORAGE) == (PackageManager.PERMISSION_GRANTED);
       return result && result1;
   }
 
   // Requesting camera permission
   private void requestCameraPermission() {
       if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.M) {
           requestPermissions(cameraPermission, CAMERA_REQUEST);
       }
   }
 
   // Requesting camera and gallery
   // permission if not given
   @Override
   public void onRequestPermissionsResult(int requestCode, @NonNull String[] permissions, @NonNull int[] grantResults) {
       super.onRequestPermissionsResult(requestCode, permissions, grantResults);
 
      switch (requestCode) {
           case CAMERA_REQUEST: {
               if (grantResults.length > 0) {
                   boolean camera_accepted = grantResults[0] == PackageManager.PERMISSION_GRANTED;
                   boolean writeStorageaccepted = grantResults[1] == PackageManager.PERMISSION_GRANTED;
                   if (camera_accepted && writeStorageaccepted) {
                       pickFromCamera();
                   } else {
                       Toast.makeText(this, "Please Enable Camera and Storage Permissions", Toast.LENGTH_LONG).show();
                   }
               }
           }
           break;
           case STORAGE_REQUEST: {
               if (grantResults.length > 0) {
                   boolean writeStorageaccepted = grantResults[0] == PackageManager.PERMISSION_GRANTED;
                   if (writeStorageaccepted) {
                       pickFromGallery();
                   } else {
                       Toast.makeText(this, "Please Enable Storage Permissions", Toast.LENGTH_LONG).show();
                   }
               }
           }
           break;
       }
   }
```






13. To pick an image from Gallery and Camera, and Display it into the imageview we created in the xml, we used several methods in MainActivity.java  to make it more readable and bug-free.
```

 // Here we will pick image from gallery or camera
   private void pickFromGallery() {
       CropImage.activity().start(MainActivity.this);
   }
   private void pickFromCamera(){
       CropImage.activity().start(MainActivity.this);
   }
 
 
 
   @Override
   protected void onActivityResult(int requestCode, int resultCode, @Nullable Intent data) {
       super.onActivityResult(requestCode, resultCode, data);
       if (requestCode == CropImage.CROP_IMAGE_ACTIVITY_REQUEST_CODE) {
           CropImage.ActivityResult result = CropImage.getActivityResult(data);
           if (resultCode == RESULT_OK) {
               Uri resultUri = result.getUri();
               Picasso.with(this).load(resultUri).into(targetframe);
           }
       }
       if (requestCode == 100) {
           // BitMap is data structure of image file which store the image in memory
           Bitmap photo = (Bitmap) data.getExtras().get("data");
           // Set the image in imageview for display
           targetframe.setImageBitmap(photo);
           Picasso.with(this).load(getImageUri(this, photo)).into(targetframe);
       }
   }
   private Uri getImageUri(Context inContext, Bitmap inImage) {
       ByteArrayOutputStream bytes = new ByteArrayOutputStream();
       inImage.compress(Bitmap.CompressFormat.JPEG, 100, bytes);
       String path = MediaStore.Images.Media.insertImage(inContext.getContentResolver(), inImage, "Title", null);
       return Uri.parse(path);
   }
 
``` 




14. To save an image from the displayed image view  to the gallery, we wrote the following method in MainActivity.java .
```
   private void imagesavetomyphonegallery() {
       Uri images;
       ContentResolver contentResolver = getContentResolver();
 
       if(Build.VERSION.SDK_INT>= Build.VERSION_CODES.Q)
       {
 
           images = MediaStore.Images.Media.getContentUri(MediaStore.VOLUME_EXTERNAL_PRIMARY);
      
 }
 
 
       else
       {
           images = MediaStore.Images.Media.EXTERNAL_CONTENT_URI;
       }
 
 
       ContentValues contentValues = new ContentValues();
       contentValues.put(MediaStore.Images.Media.DISPLAY_NAME, System.currentTimeMillis()+".jpg");
       contentValues.put(MediaStore.Images.Media.MIME_TYPE, "images/*");
       Uri uri = contentResolver.insert(images,contentValues);
 
 
       try{
           BitmapDrawable bitmapDrawable = (BitmapDrawable) targetframe.getDrawable();
           Bitmap bitmap = bitmapDrawable.getBitmap();
           OutputStream outputStream = contentResolver.openOutputStream(Objects.requireNonNull(uri));
           bitmap.compress(Bitmap.CompressFormat.JPEG, 100, outputStream);
           Objects.requireNonNull(outputStream);
           Toast.makeText(this, "Photo Saved Successfully in the Gallery!", Toast.LENGTH_LONG).show();
 
       }
 
       catch (Exception e)
       {
           e.printStackTrace();
 
       }
   }
```













## Possible Errors: 
1. You may face some errors if you don’t follow the code strictly and these are some of the reasons why:
“com.theartofdev.edmodo:android-image-cropper” Not Found 

2. In order to successfully link and use the external library, we need to add jcenter() in settings.gradle or else we will get errors.

3. Crashing when capture/save/crop button is pressed
Incase of capture, it might be action camera intent not working, but in case of save it is file permission related issue. Incase of crop capture and crop can’t always be possible, because of error in device type.


4. Could not access Gallery or Camera
This might be an error because of problems in permission, not adding permissions in the manifest and not asking in runtime might be a reason for it.


5. Not adding Intent Query
In case of not adding intent query in the android manifest, crop section camera will not load

6. Image is not saved in the gallery:
File permission is not properly given, restart the app and give file permission, in the code part, check the manifest if user permission is properly given or not.


## Application UI: 

## Github Repository: 
https://github.com/AbdullahArean/CaptureSaveCrop

## Medium Article:
https://abdullaharean.medium.com/capture-crop-and-save-image-using-andriod-studio-with-github-repository-full-code-f705302f7067 

## References: 
[1] https://developer.android.com   
[2] https://www.geeksforgeeks.org/how-to-crop-image-from-camera-and-gallery-in-android 
[3] https://guides.codepath.com/android/Accessing-the-Camera-and-Stored-Media 


