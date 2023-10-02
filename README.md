PGLIFE/api/login_submit.php
<?php
session_start();

require "../includes/database_connect_hide_error.php";

if (!isset($_SESSION['user_id'])) {
    echo json_encode(array("success" => false, "is_logged_in" => false));
    return;
}

$user_id = $_SESSION['user_id'];
$property_id = $_GET["property_id"];

$sql_1 = "SELECT * FROM interested_users_properties WHERE user_id = $user_id AND property_id = $property_id";
$result_1 = mysqli_query($con, $sql_1);
if (!$result_1) {
    echo json_encode(array("success" => false, "message" => "Something went wrong"));
    return;
}

if (mysqli_num_rows($result_1) > 0) {
  //When user has the property interested already, this block will run to delete it from being interested, upon click
  $sql_2 = "DELETE FROM interested_users_properties WHERE user_id = $user_id AND property_id = $property_id";
  $result_2 = mysqli_query($con, $sql_2);
  if (!$result_2) {
      echo json_encode(array("success" => false, "message" => "Something went wrong"));
      return;
  }
  else {
      echo json_encode(array("success" => true, "is_interested" => false, "property_id" => $property_id));
      return;
  }
}
 128 changes: 128 additions & 0 deletions128  
PGLIFE/api/login_submit.php
@@ -0,0 +1,128 @@
<?php
  session_start();
  require "../includes/database_connect_hide_error.php";

  if( $con )  //$con does not contains false or null value, i.e. $con is true
  {
    $email = $_POST["email"];
    $password = $_POST["password"];
    // If password is stored in hashed format.
    $password = sha1($password);

    $sql_query = " SELECT * FROM users WHERE email='$email' AND password='$password'; ";
    $result = mysqli_query($con,$sql_query);
    if( $result )
    {
      $row_count = mysqli_num_rows($result);

      //If user with entered email and password already exists, single record of user details will be fetched & user gets logged in, by code below:
      if( $row_count == 1 ){
        $row = mysqli_fetch_assoc($result);
        // When logged in, user_id and full_name session variables gets initialised according to the user data.
        $_SESSION["user_id"] = $row["id"];
        $_SESSION["full_name"] = $row["full_name"];
        // Upon successfull logging in, user gets redirected to home page.
        // header("location: /PGLIFE/index.php");
        // exit();
        $response = array("success" => true, "message" => "Successfully Logged IN!");
        echo json_encode($response);
        return;
      }
      elseif( $row_count == 0 ){
        //When email and password do not match/exists in record...
        $response = array("success" => false, "message" => "Incorrect Username or Password!");
        echo json_encode($response);
        return;
      }
    }
    else{
      // if $result has false value...
      $response = array("success" => false, "message" => "We couldn't log you in at the moment!");
      echo json_encode($response);
      return;
    }
  }
  else{
    //when $con has false value
    $response = array("success" => false, "message" => "Database Connectivity Error!");
    echo json_encode($response);
    return;
  }
?>


<!DOCTYPE html>
<html lang="en">

<head>
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <title>
      <?php

      ?>
    </title>

    <link href="../css/bootstrap.min.css" rel="stylesheet" />
    <link href="https://use.fontawesome.com/releases/v5.11.2/css/all.css" rel="stylesheet" />
    <link href="https://fonts.googleapis.com/css2?family=Open+Sans:ital,wght@0,300;0,400;0,600;0,700;0,800;1,300;1,400;1,600;1,700;1,800&display=swap" rel="stylesheet" />
    <link href="../css/common.css" rel="stylesheet" />
    <!-- <link href="css/index.css" rel="stylesheet" /> -->
</head>

<body>
  <!-- Header Section -->
  <?php require "../includes/header.php"; ?>

  <div class="login-content mx-4 my-5">
    <?php
    // If database connection fails
      if( !$con ){
        echo "
          <h2>Database Connectivity Error!</h2>
          <hr />
          <p>Account Creation Unsuccessfull!</p>
        ";
        echo "<h5> ".mysqli_connect_error()." </h5>";
      }
      elseif( !$result ){
        echo "
          <h2>We couldn't log you in!</h2>;
          <hr />
        ";
        echo "<h5>".mysqli_error($con)."</h5>";
      }
    ?>

    <?php
      if( $row_count == 0 )
      {
        //Code for User not able to login.
    ?>
      <div class="">
        <h2>Email or Password do not match!</h2>
        <hr>
        <p>Try again logging in with different email or password...</p>
      </div>

    <?php
      }
    ?>

  </div>


  <!-- Modal Pages -->
  <?php require "../includes/signup_modal.php"; ?>
  <?php require "../includes/login_modal.php" ?>

  <!-- Footer -->
  <?php require "../includes/footer.php" ?>

  <script type="text/javascript" src="../js/jquery.js"></script>
  <script type="text/javascript" src="../js/bootstrap.min.js"></script>
  <script type="text/javascript" src="../js/common.js"></script>
</body>

</html>

<?php mysqli_close($con); ?>
 173 changes: 173 additions & 0 deletions173  
PGLIFE/api/signup_submit.php
@@ -0,0 +1,173 @@
<?php
  // If user is already signed in, and this signup_submit.php is requested for, the user will automatically get logged out first.
  session_start();
  session_destroy();

  require "../includes/database_connect_hide_error.php";

  if( !$con )
  {
    // echo mysqli_connect_error();
    //  exit();
    $response = array("success" => false, "message" => "Database Connectivity Error!");
    echo json_encode($response);
    return;
  }
  else {
    $full_name = $_POST["full_name"];
    $phone = $_POST["phone"];
    $email = $_POST["email"];

    $password = $_POST["password"];
    // If password is to be stored in hashed format.
    $password = sha1($password);

    $college_name = $_POST["college_name"];
    if( isset($_POST["gender"]) )
      {$gender = $_POST["gender"];}

    $sql_query1 = "SELECT * FROM users where email='$email';";
    $result1 = mysqli_query($con,$sql_query1);

    if ( !$result1 ) {
      $response = array("success" => false, "message" => "Something went wrong!");
      echo json_encode($response);
      return;
    }

    //When already the email is registered by another user previosly
    $row_count = mysqli_num_rows($result1);
    if ($row_count != 0) {
      $response = array("success" => false, "message" => "This email id is already registered with us!");
      echo json_encode($response);
      return;
    }

    $result2 = FALSE;
    if( mysqli_num_rows($result1)==0 ){
      $sql_query2 = "INSERT INTO users (full_name,phone,email,password,college_name,gender) VALUES ('$full_name','$phone','$email','$password','$college_name','$gender');";
      $result2 = mysqli_query($con,$sql_query2);

      if ( !$result2 ) {
        $response = array("success" => false, "message" => "Something went wrong! Registration Unsuccessfull!");
        echo json_encode($response);
        return;
      }
      else {
        $response = array("success" => true, "message" => "User is successfully registered!");
        echo json_encode($response);
        return;
      }
    }
  }

?>

<!--
<!DOCTYPE html>
<html lang="en">
<head>
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <title>
      <?php
        // if( $result2 ){
        //   echo "Signup Successfull";
        // }
        // else{
        //   echo "Signup Error";
        // }
      ?>
    </title>
    <link href="../css/bootstrap.min.css" rel="stylesheet" />
    <link href="https://use.fontawesome.com/releases/v5.11.2/css/all.css" rel="stylesheet" />
    <link href="https://fonts.googleapis.com/css2?family=Open+Sans:ital,wght@0,300;0,400;0,600;0,700;0,800;1,300;1,400;1,600;1,700;1,800&display=swap" rel="stylesheet" />
    <link href="../css/common.css" rel="stylesheet" />
</head>
-->

<!-- <body> -->
  <!-- Header Section -->
  <?php // require "../includes/header.php"; ?>

  <!-- <div class="signup-content mx-4 my-5"> -->
    <?php
    /*
      if( !$con ){
        echo "
          <h2>Database Connectivity Error!</h2>
          <hr />
          <p>Account Creation Unsuccessfull!</p>
        ";
        echo "<h5> ".mysqli_connect_error()." </h5>";
      }
    */
    ?>

    <?php
    // When email entered by user during signup is already registered in the database
    /*
      if( mysqli_num_rows($result1)!=0 ){
    ?>
      <div class="">
        <h2>Email "<?php echo $email; ?>" is already registered with us!</h2>
        <hr />
        <p>Please sign in, or try registering with a different email account...</p>
      </div>
    <?php
      }
    */
    ?>

    <?php
    // Upon Successfull Registration...
    // PHP page to be rendered if sync request come, i.e, non-AJAX request
      // if( $result2 )
      // {
    ?>
    <!--
      <div class="">
        <h2>Dear <?php // echo $full_name ?>, your account is successfully created!</h2>
        <p>Please login to your account to continue searching for your favourite PGs!</p>
        <div class="text-center mt-3 mb-5">
          <a class="" href="#" data-toggle="modal" data-target="#login-modal">
              <i class="fas fa-sign-in-alt"></i>Login
          </a>
        </div>
      </div>
    -->

    <?php
      // }
      // When email entered is uniques but due to some reason $sql_query2 cannot be successfully executed, i.e, user details cannot be added to database
      // elseif( !$result2 && mysqli_num_rows($result1)==0 )
      // {
      //   echo "
      //     <h2>Error in Data Entry!</h2>
      //     <hr />
      //     <p>Account Creation Unsuccessfull! Try with some different data in the fields...</p>
      //   ";
      //   echo "<h5> ".mysqli_error($con)." </h5>";
      // }
    ?>
  <!-- </div> -->


  <!-- Modal Pages -->
  <?php // require "../includes/signup_modal.php"; ?>
  <?php // require "../includes/login_modal.php" ?>

  <!-- Footer -->
  <?php // require "../includes/footer.php" ?>

  <!-- <script type="text/javascript" src="../js/jquery.js"></script>
  <script type="text/javascript" src="../js/bootstrap.min.js"></script>
  <script type="text/javascript" src="../js/common.js"></script>
</body>
</html> -->

<?php mysqli_close($con); ?>
 46 changes: 46 additions & 0 deletions46  
PGLIFE/api/toggle_interested.php
@@ -0,0 +1,46 @@
<?php
session_start();

require "../includes/database_connect_hide_error.php";

if (!isset($_SESSION['user_id'])) {
    echo json_encode(array("success" => false, "is_logged_in" => false));
    return;
}

$user_id = $_SESSION['user_id'];
$property_id = $_GET["property_id"];

$sql_1 = "SELECT * FROM interested_users_properties WHERE user_id = $user_id AND property_id = $property_id";
$result_1 = mysqli_query($con, $sql_1);
if (!$result_1) {
    echo json_encode(array("success" => false, "message" => "Something went wrong"));
    return;
}

if (mysqli_num_rows($result_1) > 0) {
  //When user has the property interested already, this block will run to delete it from being interested, upon click
  $sql_2 = "DELETE FROM interested_users_properties WHERE user_id = $user_id AND property_id = $property_id";
  $result_2 = mysqli_query($con, $sql_2);
  if (!$result_2) {
      echo json_encode(array("success" => false, "message" => "Something went wrong"));
      return;
  } else {
      echo json_encode(array("success" => true, "is_interested" => false, "property_id" => $property_id));
      return;
  }
}
else {
  //When the property is not marked as interested, and upon click it is to be marked interested for the user, this block
  //of code will run...
  $sql_3 = "INSERT INTO interested_users_properties (user_id, property_id) VALUES ( $user_id, $property_id )";
  $result_3 = mysqli_query($con, $sql_3);
  if (!$result_3) {
      echo json_encode(array("success" => false, "message" => "Something went wrong"));
      return;
  }
  else {
      echo json_encode(array("success" => true, "is_interested" => true, "property_id" => $property_id));
      return;
  }
}
 1 change: 1 addition & 0 deletions1  
PGLIFE/css/bootstrap.min.css
Large diffs are not rendered by default.

 185 changes: 185 additions & 0 deletions185  
PGLIFE/css/common.css
@@ -0,0 +1,185 @@
* {
    box-sizing: border-box;
    font-family: 'Open Sans', sans-serif;
}

body {
    color: #373737;
    font-size: 16px;
    position: relative;
}

h1,
h2,
h3,
h4,
h5,
h6 {
    font-weight: 600;
}

h1 {
    font-size: 36px;
    margin-bottom: 40px;
}

@media (max-width: 768px) {
    h1 {
        font-size: 28px;
    }
}

h3 {
    font-size: 24px;
    margin-bottom: 12px;
}

h5 {
    font-size: 16px;
    margin-bottom: 10px;
}

.btn-primary,
.btn-warning,
.btn-danger {
    color: white;
    font-weight: bold;
}

.input-group .input-group-text i {
    width: 14px;
}

input[type="radio"] {
    position: relative;
    top: 1px;
}

.breadcrumb {
    background-color: #e5e3e8;
    font-size: 13px;
    border-radius: 0px;
    margin-bottom: 0px;
}

.container {
    max-width: 920px;
    padding: 0px 20px;
}

.page-container {
    max-width: 800px;
    padding: 80px 24px;
    margin: auto;
}

@media (max-width: 768px) {
    .page-container {
        padding: 64px 24px;
    }
}

.white {
    color: #ffffff;
}

#loading {
    display: none;
    background-color: #666666;
    background-image: url("../img/progress_spinner.gif");
    background-repeat: no-repeat;
    background-position: center;
    opacity: 0.4;
    position: fixed;
    top: 0px;
    right: 0px;
    width: 100%;
    height: 100%;
    z-index: 10000000;
}

/* Login Modal */
#login-modal .modal-dialog {
    width: 350px;
    margin: auto;
}

.modal-footer {
    background-color: #f4f4f4;
    justify-content: center;
}

/* Header */
.header {
    background-color: white;
    font-size: 13px;
}

.header .navbar .navbar-brand img {
    height: 40px;
}

.header .navbar .nav-item i {
    margin-right: 8px;
}

.header .nav-vl {
    border-left: 2px solid #d6d6d678;
    height: 24px;
    margin: auto 16px;
    display: block;
}

@media(max-width:768px) {
    .header .nav-vl {
        display: none;
    }
}

.header .nav-name {
    font-weight: bold;
    margin: auto 16px;
}

@media(max-width:768px) {
    .header .nav-name {
        margin: 8px 0px
    }
}

/* Footer */
.footer {
    background-color: #333333;
    font-size: 13px;
}

.footer-container {
    padding: 24px;
}

.footer-cities {
    display: flex;
    flex-wrap: wrap;
    justify-content: space-around;
}

.footer-city {
    margin-bottom: 8px;
    text-align: center;
}

@media(max-width:768px) {
    .footer-city {
        width: 100%;
    }
}

.footer-city a {
    color: #ffffff;
}

.footer-copyright {
    color: #a9a9a9;
    text-align: center;
    padding-top: 24px;
}
 33 changes: 33 additions & 0 deletions33  
PGLIFE/css/dashboard.css
@@ -0,0 +1,33 @@
/* Class Selectors */
.img-row{
  margin-top: 20px;
}

.usr-img{
  box-shadow: 0 0 3px 1px rgba(0,0,0,0.4);
  padding: 12px;
  border-radius: 50%;
}

.usr-details{
  margin-top: 20px;
  padding-top: 13px;
  line-height: 0.9;
  font-size: 0.9rem;
}

.usr-details :first-child{
  font-weight: bold;
  font-size: 1rem;
}

.edit-profile-link{
  font-size: 0.9rem;
  /* position: relative;
  top: -20px;
  left: 10px; */
}

.hide{
  display: none;
}
 68 changes: 68 additions & 0 deletions68  
PGLIFE/css/index.css
@@ -0,0 +1,68 @@
/* Tag Selectors */


/* Class Selectors */
.pg-search-container{
  background-color: black;
  background-image: linear-gradient(rgba(0, 0, 0, 0.5),
                       rgba(0, 0, 0, 0.5)), url("../img/bg.png");
  background-position-Y: -30px;
  background-attachment: fixed;
  background-size: cover;
  height: 75vh;
  padding-top: 30vh;
}

.input-group.md-form.form-sm.form-2 input {
  border: 1px solid #bdbdbd;
  border-top-left-radius: 0.25rem;
  border-bottom-left-radius: 0.25rem;
}

.input-group.md-form.form-sm.form-2 input.red-border {
  border: 1px solid #ef9a9a;
}

.city-container{
  text-align: center;
}

.city-caption{
  margin-top: 50px;
}

.city-box{
  margin-bottom: 50px;
  display: flex;
  flex-wrap: wrap;
  justify-content: center;
  align-items: center;
}

.city-img{
  border-radius: 50%;
  box-shadow: 1px 1px 5px 1px rgba(0,0,0,0.3);
  margin: 40px;
  padding: 15px;
  flex: 0 0 150px;

  transition: all 200ms ease-in-out;
}

.city-img:hover{
  box-shadow: 3px 4px 10px 3px rgba(0,0,0,0.3);
  transform: scale(1.05);
}

.city-img img{
  width: 100%;
}

@media screen and (max-width: 600px){

  .city-img{
    flex: 0 0 180px;
    margin: 40px 80px;
  }

}
 197 changes: 197 additions & 0 deletions197  
PGLIFE/css/property_detail.css
@@ -0,0 +1,197 @@
.carousel img {
    height: 300px;
    object-fit: cover;
}

.property-summary {
    padding: 24px;
}

.property-amenities,
.property-rating {
    background-color: #fcfaf7;
}

.star-container i {
    color: #EA322E;
    font-size: 12px;
    margin-right: 6px;
}

.interested-container {
    text-align: center;
    padding-right: 42px;
}

@media (max-width: 768px) {
    .interested-container {
        padding-right: 0px;
    }
}

.interested-container i {
    color: #EA322E;
    font-size: 20px;
    cursor: pointer;
}

.interested-text {
    font-size: 10px;
}

.detail-container {
    padding-bottom: 10px;
}

.property-name {
    font-size: 36px;
    font-weight: 600;
}

@media (max-width: 768px) {
    .property-name {
        font-size: 24px;
    }
}

.property-address {
    color: #6f6f6f;
    font-size: 16px;
    padding-bottom: 5px;
}

.property-gender img {
    width: 40px;
}

.rent-container {
    display: flex;
    align-items: center;
}

.rent {
    font-size: 24px;
    font-weight: bold;
    padding-right: 10px;
}

@media (max-width: 768px) {
    .rent {
        font-size: 18px;
    }
}

.rent-unit {
    color: #6f6f6f;
    font-size: 12px;
}

.button-container .btn {
    font-size: 14px;
    border-radius: 0px;
    width: 140px;
    float: right;
}

@media (max-width: 768px) {
    .property-amenities .row>div {
        padding-bottom: 24px;
    }
}

.amenity-container {
    margin: 8px 0px;
}

.amenity-container img {
    height: 20px;
    margin-top: -4px;
    margin-right: 4px;
}

.rating-criteria {
    margin: 28px 0px;
}

.rating-criteria-icon {
    width: 14px;
}

.rating-criteria-text {
    margin-left: 8px;
}

.rating-criteria-star-container {
    margin-top: -2px;
}

.rating-criteria-star-container i {
    color: #66C2BD;
    font-size: 12px;
}

.rating-circle {
    background-color: #66C2BD;
    color: white;
    text-align: center;
    height: 160px;
    width: 160px;
    border-radius: 50%;
    padding-top: 32px;
    margin: auto;
}

.total-rating {
    font-size: 40px;
}

.rating-circle-star-container {
    font-size: 12px;
}

.testimonial-block {
    background-color: #fcfaf7;
    padding: 0px 160px 24px;
    margin-top: 80px;
}

@media (max-width: 768px) {
    .testimonial-block {
        padding: 0px 24px 24px;
    }
}

.testimonial-image-container {
    text-align: center;
}

.testimonial-img {
    width: 100px;
    border-radius: 50%;
    position: relative;
    top: -40px;
}

.testimonial-text {
    color: #777777;
    text-align: center;
    position: relative;
}

.testimonial-text i {
    font-size: 20px;
    position: absolute;
    left: 0px;
    top: 0px;
}

.testimonial-text p {
    text-indent: 20px;
}

.testimonial-name {
    color: #2F2E2E;
    text-align: right;
    font-weight: bold;
    margin-top: 12px;
}
 154 changes: 154 additions & 0 deletions154  
PGLIFE/css/property_list.css
@@ -0,0 +1,154 @@
.filter-bar > div {
    cursor: pointer;
    font-size: 13px;
    text-align: center;
    margin-bottom: 16px;
}

.filter-bar img {
    width: 40px;
    height: 40px;
    border-radius: 40px;
    border: 1px solid #646870;
    padding: 6px;
    margin-right: 8px;
}

@media (max-width: 768px) {
    .filter-bar img {
        display: block;
        margin: 0px auto 4px;
    }
}

.property-card {
    background-color: #ffffff;
    border-radius: 2px;
    padding: 15px 0px;
    margin: 0px auto 20px;
    box-shadow: 0 2px 5px 0 rgba(0, 0, 0, 0.16), 0 2px 10px 0 rgba(0, 0, 0, 0.12);
}

.property-card:focus,
.property-card:hover {
    box-shadow: 0 5px 11px 0 rgba(0, 0, 0, 0.18), 0 4px 15px 0 rgba(0, 0, 0, 0.15);
}

.image-container {
    text-align: center;
}

@media (max-width: 768px) {
    .image-container {
        padding-bottom: 12px;
    }
}

.image-container img {
    width: 100%;
    max-width: 300px;
}

.star-container i {
    color: #EA322E;
    font-size: 10px;
    margin-right: 6px;
}

.interested-container {
    text-align: center;
}

.interested-container i {
    color: #EA322E;
    font-size: 20px;
    cursor: pointer;
}

.interested-text {
    font-size: 10px;
}

.detail-container {
    padding-bottom: 10px;
}

.property-name {
    font-size: 18px;
    font-weight: 600;
}

.property-address {
    color: #6f6f6f;
    font-size: 13px;
    padding-bottom: 5px;
}

.property-gender img {
    width: 40px;
}

.rent-container {
    display: flex;
    align-items: center;
}

.rent {
    font-size: 18px;
    font-weight: bold;
    padding-right: 10px;
}

.rent-unit {
    color: #6f6f6f;
    font-size: 12px;
}

.button-container .btn {
    font-size: 14px;
    border-radius: 0px;
    width: 140px;
    float: right;
}

/* No Property */
.no-property-container {
    margin: 96px auto 144px;
}

.no-property-container p {
    font-size: 24px;
    text-align: center;
}

/* Filter Modal */
#filter-modal .modal-dialog {
    max-width: 600px;
}

#filter-modal .modal-body h5 {
    color: #777777;
}

#filter-modal .modal-body hr {
    margin: 4px 0px;
}

#filter-modal .modal-body button {
    font-size: 14px;
    width: 100px;
    margin: 16px;
}

#filter-modal .modal-body .btn-active {
    color: #fff;
    background-color: #343a40;
}

#filter-modal .modal-body button i {
    margin-right: 8px;
}

#filter-modal .modal-footer button {
    width: 120px;
}
 282 changes: 282 additions & 0 deletions282  
PGLIFE/dashboard.php
@@ -0,0 +1,282 @@
<?php
  session_start();

  // If user is logged in, only then user can view the dashboard.php page, else will get redirected to homepage.
  if( isset($_SESSION["user_id"]) )
  {
    require "includes/database_connect.php";
    if( !$con )
    {
      echo "Couldn't connect to database!\n";
      echo mysqli_connect_error();
    }
    else
    {
      $user_id = $_SESSION["user_id"];
      $sql_query = "SELECT * FROM users WHERE id='$user_id';";
      $result = mysqli_query($con,$sql_query);
      if( !$result )
      {
        echo "Couldn't Authenticate User!";
        echo mysqli_error($con);
      }
      else
      {
        $row = mysqli_fetch_assoc($result);
        $full_name = $_SESSION["full_name"];
        $email = $row["email"];
        $phone = $row["phone"];
        $college = $row["college_name"];

        //Fetching liked properties of the logged in user...
        $sql_liked_property = "SELECT * FROM interested_users_properties INNER JOIN properties ON interested_users_properties.property_id = properties.id WHERE user_id=$user_id; ";
        $liked_property_result =  mysqli_query($con, $sql_liked_property);

        if( !$liked_property_result ){
          echo mysqli_error($con);
        }
      }
    }
  }
  else
  {
    header("location: index.php");
    exit();
  }

?>

<!DOCTYPE html>
<html lang="en">

<head>
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <title>Best PG's in Mumbai | PG Life</title>

    <link href="css/bootstrap.min.css" rel="stylesheet" />
    <link href="https://use.fontawesome.com/releases/v5.11.2/css/all.css" rel="stylesheet" />
    <link href="https://fonts.googleapis.com/css2?family=Open+Sans:ital,wght@0,300;0,400;0,600;0,700;0,800;1,300;1,400;1,600;1,700;1,800&display=swap" rel="stylesheet" />
    <link href="css/common.css" rel="stylesheet" />
    <link href="css/property_list.css" rel="stylesheet" />
    <link href="css/dashboard.css" rel="stylesheet" />
</head>

<body>
  <!-- Header Section -->
  <?php require "./includes/header.php"; ?>

  <div id="loading">
  </div>

  <nav aria-label="breadcrumb">
      <ol class="breadcrumb py-2">
          <li class="breadcrumb-item">
              <a href="index.php">Home</a>
          </li>
          <li class="breadcrumb-item active" aria-current="page">
              Dashboard
          </li>
      </ol>
  </nav>


  <div class="container my-5">
    <div class="row mt-5">
      <div class="col-sm-5 offset-sm-2">
        <h1>My Profile</h1>
      </div>
    </div>
    <div class="row justify-content-sm-center">
      <div class="col-sm-3 img-row ">
        <img src="./img/user.png" alt="User Image" class="usr-img">
      </div>
      <div class="col-sm-5 usr-details">
        <p><?php echo $full_name; ?></p>
        <p><?php echo $email; ?></p>
        <p><?php echo $phone; ?></p>
        <p><?php echo $college; ?></p>
      </div>
    </div>
    <div class="row justify-content-sm-end">
      <div class="col-3 offset-9">
        <a href="#" class="delete-profile-link" id="del-link">Delete Profile</a>
      </div>
    </div>
  </div>


  <div class="page-container">

    <div class="property-heading ">
      <h2>My Interested Properties:</h2>
      <hr>
    </div>
      <!-- List of city cards -->
      <?php
        if( $liked_property_result && mysqli_num_rows($liked_property_result)>0 )   //When the user has some liked properties...
        {
      ?>

          <?php

          //Loop to show each property one-by-one in the page
          while( $property_row = mysqli_fetch_assoc($liked_property_result) )
          {
            $pg_id = $property_row["id"];
            $pg_name = $property_row["name"];
            $pg_address = $property_row["address"];
            $gender_allowed = $property_row["gender"];
            $pg_rent = $property_row["rent"];
            $pg_rating_overall = round(( $property_row["rating_clean"] + $property_row["rating_food"] + $property_row["rating_safety"] )/3 , 1 );

            //Fetching the number of person that marked the property interested...
            $sql_interest = "SELECT * FROM interested_users_properties WHERE property_id = $pg_id; ";
            $interest_result = mysqli_query($con,$sql_interest);
            if( $interest_result ){
              $num_interested = mysqli_num_rows($interest_result);
            }
      ?>
            <!-- Property Card containing details of single PG -->
            <div class="property-card row" id="card-<?php echo $pg_id; ?>">
                <div class="image-container col-md-4">
                    <img src="img/properties/1/1d4f0757fdb86d5f.jpg" />
                </div>
                <div class="content-container col-md-8">
                    <div class="row no-gutters justify-content-between">
                        <div class="star-container" title="<?php echo $pg_rating_overall; ?>">
                            <!--
                            <i class="fas fa-star"></i>             - full coloured star
                            <i class="fas fa-star-half-alt"></i>    - half coloured star
                            <i class="far fa-star"></i>             - full empty start
                          -->
                          <?php
                            $star_full = floor($pg_rating_overall);
                            $star_empty = 5-1-$star_full;
                            //Full coloured star loop
                            for( $i=1 ; $i<=$star_full ; $i++ ){
                              ?> <i class="fas fa-star"></i> <?php
                            }

                            // Half coloured star placeholder
                            if( ($pg_rating_overall - $star_full)>0.2 && ($pg_rating_overall - $star_full)<0.8 ){
                              ?> <i class="fas fa-star-half-alt"></i> <?php
                            }
                            elseif( ($pg_rating_overall - $star_full)>=0.8 ){
                              ?> <i class="fas fa-star"></i> <?php
                            }
                            elseif( ($pg_rating_overall - $star_full)<=0.2 ){
                              ?> <i class="far fa-star"></i> <?php
                            }

                            //Empty coloured star loop
                            for( $i=1 ; $i<=$star_empty ; $i++ ){
                              ?> <i class="far fa-star"></i> <?php
                            }

                          ?>
                        </div>



                        <div class="interested-container <?php echo "property-id-" . $pg_id; ?>">
                          <?php
                          //During 1st time loading of page, checking if property is already marked interested by the logged in user, or not.
                          //This check is done to

                          if( isset($_SESSION["user_id"]) ){
                            //User is logged in...
                            $user_id = $_SESSION["user_id"];
                            $sql_isLiked = "SELECT * FROM interested_users_properties WHERE user_id = $user_id AND property_id = $pg_id";
                            $isLiked_result = mysqli_query($con, $sql_isLiked);
                          }
                          ?>
                          <?php
                            if( isset($_SESSION["user_id"]) && mysqli_num_rows($isLiked_result)==1 ){
                          ?>
                              <i class="is-interested-image fas fa-heart" property_id=<?php echo $pg_id; ?> ></i>
                          <?php
                            }
                            else{
                          ?>
                              <i class="is-interested-image far fa-heart" property_id=<?php echo $pg_id; ?> ></i>
                          <?php
                            } ?>

                            <div class="interested-text">
                              <?php if( $interest_result ){ ?>
                                <span class="interested-user-count"><?php echo $num_interested; ?></span> interested
                              <?php } ?>
                            </div>
                        </div>


                    </div>
                    <div class="detail-container">
                        <div class="property-name"><?php echo $pg_name; ?></div>
                        <div class="property-address"><?php echo $pg_address; ?></div>
                        <div class="property-gender">

                            <?php
                              if( $gender_allowed=='male' ){
                                ?> <img src="img/male.png" /> <?php
                              }
                              elseif( $gender_allowed=='female' ){
                                ?> <img src="img/female.png" /> <?php
                              }
                              else{
                                ?> <img src="img/unisex.png" /> <?php
                              }
                            ?>

                        </div>
                    </div>
                    <div class="row no-gutters">
                        <div class="rent-container col-6">
                            <div class="rent">Rs <?php echo $pg_rent; ?>/-</div>
                            <div class="rent-unit">per month</div>
                        </div>
                        <div class="button-container col-6">
                            <a href="property_detail.php?property_id=<?php echo $pg_id ;?>" class="btn btn-primary">View</a>
                        </div>
                    </div>
                </div>
            </div>

      <?php
          }
        }
        elseif( mysqli_num_rows($liked_property_result)==0 )
        {
          //Code for user still not having any property marked as interested....
          ?>
          <div class="no-pg mx-4 text-center">
            <p><span style="font-size: 2rem; ">🙅</span></p>
            <p>You haven't marked any PG as interested yet...</p>
          </div>
          <?php
        }

      ?>



  </div>


  <!-- Modal Pages -->
  <?// php require "./includes/signup_modal.php"; ?>
  <?// php require "./includes/login_modal.php" ?>


  <!-- Footer -->
  <?php require "./includes/footer.php" ?>

  <script type="text/javascript" src="js/jquery.js"></script>
  <script type="text/javascript" src="js/bootstrap.min.js"></script>
  <!-- <script type="text/javascript" src="js/common.js"></script> -->
  <script type="text/javascript" src="js/dashboard.js"></script>
</body>

</html>

<?php mysqli_close($con); ?>
