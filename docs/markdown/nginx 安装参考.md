 ./configure --add-module=../nginx-upload-module-2.3.0 --add-module=../nginx-upload-progress-module-0.9.2
make
make install


location /upload {  
    upload_pass     /;  
    # upload_cleanup 400 404 499 500-505;  
    upload_store    /app/word/;  
    upload_store_access user:rw;  
    # upload_limit_rate 128k;  
    upload_set_form_field "${upload_field_name}_name" $upload_file_name;  
    upload_set_form_field "${upload_field_name}_content_type" $upload_content_type;  
    upload_set_form_field "${upload_field_name}_path" $upload_tmp_path;  
    upload_aggregate_form_field "${upload_field_name}_md5" $upload_file_md5;  
    upload_aggregate_form_field "${upload_field_name}_size" $upload_file_size;  
    upload_pass_form_field "^.*$";  
}