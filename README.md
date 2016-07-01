# Get-Image-From-Gallery



1). Put below code to access gallery(For Example : OnclickListener of view)

		if (Build.VERSION.SDK_INT == Build.VERSION_CODES.KITKAT) {

            intent = new Intent(Intent.ACTION_GET_CONTENT);
            intent.addCategory(Intent.CATEGORY_OPENABLE);
            intent.setType("image/*");
            startActivityForResult(Intent.createChooser(intent, "Select File"), GALLERY_IMAGE);

        } else if (Build.VERSION.SDK_INT == Build.VERSION_CODES.M) {

            String permission = Common.permissionStorage.getString("storage", "");

            if (permission.equals("true")) {

                intent = new Intent(Intent.ACTION_PICK,
                        MediaStore.Images.Media.EXTERNAL_CONTENT_URI);
                intent.setType("image/*");
                startActivityForResult(Intent.createChooser(intent, "Select File"), GALLERY_IMAGE);

            } else {

                checkPermission();

            }
        } else {

            intent = new Intent(Intent.ACTION_PICK,
                    android.provider.MediaStore.Images.Media.EXTERNAL_CONTENT_URI);
            intent.setType("image/*");
            startActivityForResult(Intent.createChooser(intent, "Select File"), GALLERY_IMAGE);

        }
        
====================================================================================================================
        
2). Now onActivityResult use following code to get selected imagePath(dataType : String) and Bitmap(dataType : Bitmap) from   that imagePath use following

    private Bitmap getSelectedImageFromGallery(Activity activity, Intent data) {
        
        Bitmap selected_bimap = null;

        Uri selectedImageUri = data.getData();

        String[] projection = {MediaStore.MediaColumns.DATA};

        Cursor cursor = activity.getContentResolver().query(selectedImageUri, projection, null,
                null, null);

        int column_index = cursor.getColumnIndexOrThrow(MediaStore.MediaColumns.DATA);

        cursor.moveToFirst();

        String selectedImagePath = cursor.getString(column_index);

        BitmapFactory.Options options = new BitmapFactory.Options();
        options.inJustDecodeBounds = true;

        BitmapFactory.decodeFile(selectedImagePath, options);

        final int REQUIRED_SIZE = 1080;

        int scale = 1;

        while (options.outWidth / scale / 2 >= REQUIRED_SIZE &&
                options.outHeight / scale / 2 >= REQUIRED_SIZE)
            scale *= 2;

        options.inSampleSize = scale;
        options.inJustDecodeBounds = false;

        return (selected_bimap = BitmapFactory.decodeFile(selectedImagePath, options));

    }
    
  for Kitkat version use bellow code...
 
     private Bitmap getImageFromKitkat(Activity activity, Intent data) {

        ParcelFileDescriptor parcelFileDescriptor = null;

        try {

            parcelFileDescriptor = activity.getContentResolver().
                    openFileDescriptor(data.getData(), "r");

            FileDescriptor fileDescriptor = parcelFileDescriptor.getFileDescriptor();

            BitmapFactory.Options options = new BitmapFactory.Options();
            options.inJustDecodeBounds = true;

            BitmapFactory.decodeFileDescriptor(fileDescriptor, null, options);

            final int REQUIRED_SIZE = 700;

            int scale = 1;

            while (options.outWidth / scale / 2 >= REQUIRED_SIZE
                    && options.outHeight / scale / 2 >= REQUIRED_SIZE)
                scale *= 2;

            options.inSampleSize = scale;
            options.inJustDecodeBounds = false;

            Bitmap bmp = BitmapFactory.decodeFileDescriptor(fileDescriptor, null, options);

            parcelFileDescriptor.close();

            return bmp;

        } catch (Exception e) {

            System.out.print("KitkatGalleryException ...>>>..." + e.getMessage());

            return null;

        }
    }
    
3).Only for marshmallow devices to check permission programmatically use below code...

    private void checkPermission() {

        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.M) {

            int hasWritePermission = checkSelfPermission(Manifest.permission.
                    WRITE_EXTERNAL_STORAGE);
            int hasReadPermission = checkSelfPermission(Manifest.permission.
                    READ_EXTERNAL_STORAGE);
            int hasCameraPermission = checkSelfPermission(Manifest.permission.CAMERA);

            List<String> permissions = new ArrayList<String>();
            if (hasWritePermission != PackageManager.PERMISSION_GRANTED) {

                permissions.add(Manifest.permission.WRITE_EXTERNAL_STORAGE);

            } else {

                Common.editSharedPreferences("storage", "true");

            }

            if (hasReadPermission != PackageManager.PERMISSION_GRANTED) {

                permissions.add(Manifest.permission.READ_EXTERNAL_STORAGE);

            } else {

                Common.editSharedPreferences("storage", "true");

            }

            if (hasCameraPermission != PackageManager.PERMISSION_GRANTED) {

                permissions.add(Manifest.permission.CAMERA);

            } else {

                Common.editSharedPreferences("camera", "true");

            }

            if (!permissions.isEmpty()) {

                requestPermissions(permissions.toArray(new String[permissions.size()]), 111);

            }
        }
    }
    
    @Override
    public void onRequestPermissionsResult(int requestCode, String[] permissions, int[] grantResults) {

        switch (requestCode) {

            case 111: {

                for (int i = 0; i < permissions.length; i++) {

                    if (grantResults[i] == PackageManager.PERMISSION_GRANTED) {

                        System.out.println("Permissions --> " + "Permission Granted: " +
                                permissions[i]);

                        if (permissions[i].equals(Manifest.permission.WRITE_EXTERNAL_STORAGE) ||
                                permissions[i].equals(Manifest.permission.READ_EXTERNAL_STORAGE)) {

                            editSharedPreferences("storage", "true");

                        } else if (permissions[i].equals(Manifest.permission.CAMERA)) {

                            editSharedPreferences("camera", "true");

                        }
                    } else if (grantResults[i] == PackageManager.PERMISSION_DENIED) {

                        System.out.println("Permissions --> " + "Permission Denied: " +
                                permissions[i]);

                        if (permissions[i].equals(Manifest.permission.WRITE_EXTERNAL_STORAGE) ||
                                permissions[i].equals(Manifest.permission.READ_EXTERNAL_STORAGE)) {

                            editSharedPreferences("storage", "false");

                        } else if (permissions[i].equals(Manifest.permission.CAMERA)) {

                            editSharedPreferences("camera", "false");

                        }
                    }
                }
            }

            break;

            default: {

                super.onRequestPermissionsResult(requestCode, permissions, grantResults);

            }
        }
    }
    
    private void editSharedPreferences(String tag, String value) {

        SharedPreferences.Editor editPermissions = permissionStorage.edit();
        editPermissions.putString(tag, value);
        editPermissions.commit();

    }
