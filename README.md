###################
<h1> Enrollment Update </h1>
###################

<pre>Retrieve data from one database and insert that data into another database, 
both databases are on the same server</pre>

<pre><?php

/**
* @author Manton Horton 
* Retrieve data from the sedb database  
* and insert that data into the bgdb database
* for customer service reasons
*
*/

$r1 = array();

$cad = array();

$prefixarray = array();

$host = $_SERVER['HTTP_HOST'];

$datab  = substr($host, 0, strpos($host, '.'));

// Put the correct login credentials
if( $datab == 'dev-training'){

                              //dev database connection -----> Hostname, Username, Password, Database Name
  $handle  =  mysqli_connect("usev-dev-db.cth3u1ikewqq.us-east-1.rds.amazonaws.com", "inetstaging", "root","db_name");

                              //moodle database connection -----> Hostname, Username, Password, Database Name
  $handlemoodle  =  mysqli_connect("usev-dev-db.cth3u1ikewqq.us-east-1.rds.amazonaws.com", "inetstaging", "root","db_name");

}elseif( $datab =='training'){

                              //production database connection -----> Hostname, Username, Password, Database Name
  $handle  =  mysqli_connect("usev-dev-db.cth3u1ikewqq.us-east-1.rds.amazonaws.com", "inetstaging", "root","db_name");

                              //moodle database connection -----> Host, Username, Password, Database Name
  $handlemoodle  =  mysqli_connect("usev-dev-db.cth3u1ikewqq.us-east-1.rds.amazonaws.com", "inetstaging", "root","db_name");

}else{

    die(mysqli_error());

}

// $handle  =  mysqli_connect("localhost", "root", "root") or die(mysqli_error());

function stage1(){
    // global variables  
    global $cad;
    global $r1;
    global $handle;
    global $handlemoodle;
    global $prefixarray;


    /* retrieve data from sedb - online training database for group enrollment using date_transmitted column to filter results */
    $sql = "SELECT * FROM bgea_churchgroup_enrollment WHERE ISNULL(`date_transmitted`)";
    
    // execute sql statement
    $result = mysqli_query($handlemoodle, $sql);

      // while loop through the results and parse the results object
      while ($data = mysqli_fetch_object($result)){

          $r1[] = $data->id;

          $variable1 = $data->firstname;

          $variable2 = $data->lastname;

          $variable3 = $data->email;

          $variable4 = $data->city;

          $variable5 = $data->zip;

          $variable6 = $data->date_created;
    
          // insert data from sedb database and into bgdb(database) / bgea-ds-vic-common-data(table) 
          mysqli_query($handle, "INSERT INTO bgea_ds_vic_common_data SET 
                  first_name = '$variable1',
                  last_name = '$variable2',
                  email_address = '$variable3',
                  city = '$variable4',
                  collection_type_id = 4,
                  source_code = 'BY000INTR',
                  vic_collection_type_id = 1,
                  postal_code = '$variable5'");

           if (!mysqli_query($handle, $sql)) {

            exit('aborted');

          }
          // insert commom_answer_id into an array and store it for referencing in the following stages
          $cad[] = $handle->insert_id;

          // 2-dimensional array insert product_code prefixes
          $prefixarray[$handle->insert_id] = array_shift(explode('-',$data->coursecatidnumber));

      }
      // calls stage2() function    
      stage2();
}

function stage2(){
  // global variables
  global $cad;
  global $handle;
  global $prefixarray;

  // local variables
  $c2 = implode(',', $cad);

    /* transfer data from two different data tables bgea-ds-vic-common-data(billygraham.org) and bgea-ds-transmit-interface(billygrahm.org), by filtering results according to the date functions and transfer the data by referencing the common-answer-id */
    $sql = "SELECT * FROM bgea_ds_vic_common_data WHERE common_answer_id IN ($c2)";

      // execute sql statement
      $result = mysqli_query($handle, $sql);
     
      // while loop through the results and parse the results object
      while ($data = mysqli_fetch_object($result)){

          $variable1 = $data->common_answer_id;

          $variable2 = $data->collection_type_id;

          $variable3 = $data->first_name;

          $variable4 = $data->last_name;

          $variable5 = $data->address_1;

          $variable6 = $data->city;

          $variable7 = $data->state;

          $variable8 = $data->postal_code;

          $variable9 = $data->email_address;

          // insert data from bgea-ds-vic-common-data(table) into the bgea-ds-transmit-interface(table)
          mysqli_query($handle, "INSERT INTO bgea_ds_transmit_interface SET 
              common_answer_id = '$variable1',
              collection_type_id = '$variable2',
              first_name = '$variable3',
              last_name = '$variable4',
              address1 = '$variable5',
              city = '$variable6',
              state = '$variable7',
              postal_code = '$variable8',
              source_code = 'BY000INTR',
              transaction_code = 'WEBDONS',
              vic_collection_type_id = 1,
              email_address = '$variable9'");     
     
      }
      // calls stage3() function 
      stage3();
}

function stage3(){ 
  // global variables  
  global $cad;
  global $handle;
  global $prefixarray;

  // local variables
  $code_value = '';
  $c2 = implode(',', $cad);
 
      /* transfer data from two different data tables bgea-ds-vic-common-data(billygraham.org) and bgea_ds_transmit_product(billygrahm.org), by filtering results according to the date functions and transfer the data by referencing the common-answer-id */
      $sql = "SELECT * FROM bgea_ds_vic_common_data WHERE common_answer_id IN ($c2)";

      // execute sql statement
      $result = mysqli_query($handle, $sql);
    
      // while loop through the results and parse the results object
      while ($data = mysqli_fetch_object($result)){

          $variable1 = $data->common_answer_id;

          $variable2 = $data->collection_type_id;

          $variable3 = $data->first_name;

          $variable4 = $data->last_name;

          $variable5 = $data->address_1;

          $variable6 = $data->city;

          $variable7 = $data->state;

          $variable8 = $data->postal_code;

          $variable9 = $data->email_address;
              
          
          /* convert the code_value using a conditional and the 2 dimensional array for common_answer_id */
          if($prefixarray[$variable1] == 'REI'){

              $code_value = P100;

          }elseif($prefixarray[$variable1] == 'SHIC'){

              $code_value = P300;

          }elseif($prefixarray[$variable1] == 'SEO'){

              $code_value = P200;

          // }elseif($prefixarray[$variable1] == 'BTTB'){

          //     $code_value = B256;

          }elseif($prefixarray[$variable1] == 'SHICST'){

              $code_value = P400;

          }

          
          // echo ' common_answer_id: '.$variable1.' code_value: '.$code_value;
          // echo '<br>';
          

          // insert data from bgea-ds-vic-common-data(table) into the bgea-ds-transmit-product(table)          
          mysqli_query($handle, "INSERT INTO bgea_ds_transmit_codes SET 
                    common_answer_id = '$variable1',
                    code_type = 'SPECIAL',
                    code_value = '$code_value'");
 
      }
      // calls stage4() function 
      stage4();   
}

function stage4(){ 
  // global variables  
  global $cad;
  global $handle;
  global $prefixarray;

  // local variables
  $c2 = implode(',', $cad);
 
      /* transfer data from two different data tables bgea-ds-vic-common-data(billygraham.org) and bgea_ds_transmit_product(billygrahm.org), by filtering results according to the date functions and transfer the data by referencing the common-answer-id */
      $sql = "SELECT * FROM bgea_ds_vic_common_data WHERE common_answer_id IN ($c2)";

      // execute sql statement
      $result = mysqli_query($handle, $sql);
    
      // while loop through the results and parse the results object
      while ($data = mysqli_fetch_object($result)){

          $variable1 = $data->common_answer_id;

          $variable2 = $data->collection_type_id;

          $variable3 = $data->first_name;

          $variable4 = $data->last_name;

          $variable5 = $data->address_1;

          $variable6 = $data->city;

          $variable7 = $data->state;

          $variable8 = $data->postal_code;

          $variable9 = $data->email_address;
              
        
          // insert data from bgea-ds-vic-common-data(table) into the bgea-ds-ecommerce-data(table)          
          mysqli_query($handle, "INSERT INTO bgea_ds_ecommerce_data SET 
                    common_answer_id = '$variable1',
                    ecommerce_type_id = 1,
                    total_amount = 0,
                    tax_amount = 0,
                    tax_rate = 0,
                    shipping_amount = 0,
                    is_recurring = 0");
 
      }
      // calls stage5() function 
      stage5();   
}

function stage5(){
  // global variables
  global $r1;
  global $handlemoodle;
  
  // local variable
  $r2 = implode(',', $r1);

  // DateTime object
  $current = new DateTime('now');
  $current_now = $current->format('Y-m-d H:i:s');
  

  // update transmitted date from bgea_churchgroup_enrollment(table) where it is NULL
  $sql = "UPDATE bgea_churchgroup_enrollment SET date_transmitted='$current_now' WHERE id IN ($r2)";

  // execute sql statement
  $result = mysqli_query($handlemoodle, $sql);  
        
}

// calls stage1() function 
stage1();

// exit("Job Done!");

?></pre>


