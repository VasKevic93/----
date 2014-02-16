<?php
error_reporting(0);

include('conf.php');

$site=$_SERVER['HTTP_HOST'];

@mysql_query('set character_set_client="cp1251"');
@mysql_query('set character_set_results="cp1251"');
@mysql_query('set collation_connection="cp1251_general_ci"');


ini_set('session.use_cookies', 'On');
ini_set('session.use_trans_sid', 'Off');
session_set_cookie_params(0, '/');

session_start();

$time=time()+$time_move*3600;
$start_time=strtotime($start_data);
$work_time=floor(($time-$start_time)/(1*3600));


if($start_time-$time<=0){
if($d_isum!=0){
$d_max=$d_max+$d_isum*floor(($time-$start_time)/($d_itime*3600));
if($d_max>$d_istop){ $d_max=$d_istop; }
}
}


// ======================================== IP ====================================================================================

if(getenv('HTTP_CLIENT_IP') && strcasecmp(getenv('HTTP_CLIENT_IP'),'unknown'))
$ip=getenv('HTTP_CLIENT_IP');
elseif(getenv('HTTP_X_FORWARDED_FOR') && strcasecmp(getenv('HTTP_X_FORWARDED_FOR'), 'unknown'))
$ip=getenv('HTTP_X_FORWARDED_FOR');
elseif(getenv('REMOTE_ADDR') && strcasecmp(getenv("REMOTE_ADDR"), 'unknown'))
$ip=getenv('REMOTE_ADDR');
elseif(!empty($_SERVER['REMOTE_ADDR']) && strcasecmp($_SERVER['REMOTE_ADDR'], 'unknown'))
$ip=$_SERVER['REMOTE_ADDR'];
else{$ip='unknown';}


// ======================================== ПЕРЕХОД РЕФЕРАЛА ====================================================================================

if(!empty($_GET['ref'])){
session_unset();
$_GET['ref']=preg_replace("#[^a-z\_\-0-9]+#i",'',$_GET['ref']);
if($_GET['ref']!=''){
$refq=mysql_query("SELECT login FROM users WHERE login='".$_GET['ref']."'");
if(mysql_num_rows($refq)>0){
$refm=mysql_fetch_row($refq);
$_SESSION['ref_login']=$refm[0];
}
}
}

// ======================================== АВТОРИЗАЦИЯ ====================================================================================

define('SID',session_id());

// ФУНКЦИЯ ВХОДА

function login($uname,$qiwi){
$q=mysql_query("SELECT uid,login,qiwi,ref FROM users WHERE login='$uname' AND qiwi='$qiwi';");
$user=mysql_fetch_row($q);

if(!empty($user)) {
session_unset();
$_SESSION['uid']=$user[0];
$_SESSION['login']=$user[1];
$_SESSION['qiwi']=$user[2];
$_SESSION['ref']=$user[3];
$_SESSION['can']=1;
return true;
}
else{
return false;
}
}

//  ЕСЛИ АВТОРИЗОВАН, БЕРЁМ АНКЕТУ ИЗ СЕССИИ

if(!empty($_SESSION['uid']) && !empty($_SESSION['login']) && !empty($_SESSION['qiwi']) && isset($_SESSION['ref'])) {
define('USER_LOGGED',true);
$u_id=$_SESSION['uid'];
$u_login=$_SESSION['login'];
$u_qiwi=$_SESSION['qiwi'];
$u_ref=$_SESSION['ref'];
}
else { define('USER_LOGGED',false); }


// ПРИЁМ ДАННЫХ ИЗ ФОРМЫ

if (!empty($_POST['login']) && !empty($_POST['qiwi'])) {
$_POST['login']=preg_replace("#[^a-z\_\-0-9]+#i",'',$_POST['login']);
$_POST['qiwi']=preg_replace('#[^a-zA-Z\-\_0-9]+#','',$_POST['qiwi']);
$_POST['qiwi']=md5($_POST['qiwi']);
if(login($_POST['login'],$_POST['qiwi'])){ header('Refresh: 0'); exit; }
else{ $wrong_lq=1; }
}


// ========================================  ОНЛАЙН ЮЗЕРЫ  ==============================================================================

function count_online($ip,$time){
if($ip!='unknown'){
$ip=preg_replace("#[^0-9]+#i",'',$ip);
$last_time=$time+20*60;
$result=mysql_query("SELECT last_time FROM online WHERE ip='$ip'");
if(mysql_num_rows($result)>0){ mysql_query("UPDATE online SET last_time=$last_time WHERE ip='$ip' LIMIT 1"); }
else{ mysql_query("INSERT INTO online (ip,last_time) VALUES ('$ip',$last_time)"); }
mysql_query('DELETE FROM online WHERE last_time<'.$time);
}
return mysql_num_rows(mysql_query('SELECT * FROM online'));
}


$dataq=mysql_query("SELECT * FROM data");
$d=mysql_fetch_row($dataq);

$d_users=$d[0];
$d_vklad=$d[1];
$d_popolnenie=$d[2];
$d_vyvod=$d[3];
$d_premod_r=$d[4];
$d_count_r=$d[5];
$d_plus=$d[6];
$d_with=$d[7];
$d_plus_n=$d[8];
$d_with_n=$d[9];
$d_new_u=$d[10];

$free=$d_plus-$d_with-($d_plus*($tocom/100));

if($start_time-$time>0){
$d_vklad=1;
$d_popolnenie=1;
$d_vyvod=1;
}

$uu=substr($d_users,-2);
$ux1=array(2,3,4,22,23,24,32,33,34,42,43,44,52,53,54,62,63,64,72,73,74,82,83,84,92,93,94);
$ux2=array(1,21,31,41,51,61,71,81,91);
$ut='Участников';
if(in_array($uu,$ux1)){ $ut='Участника'; }
elseif(in_array($uu,$ux2)){ $ut='Участник'; }



?>
<!DOCTYPE html>
<html xmlns="http://www.w3.org/1999/xhtml" >
<head>
<meta http-equiv="content-type" content="text/html; charset=windows-1251" />
<title>PROFIT-QIWI КИВИ-Умножитель</title>
<link rel="icon" href="images/icon.png" type="image/x-icon" />
<link rel="shortcut icon" href="images/icon.png"/>
<link rel="stylesheet" href="css/style.css" type="text/css" />
<link rel="stylesheet" href="css/pages.css" type="text/css" />
<link rel="stylesheet" href="css/cabinet.css" type="text/css" />
<script type="text/javascript" src="js.js"></script>
</head>
<body>

<br>

<table class="index_top" align="center" cellpadding="0px" cellspacing="0px">
<tr>
<td height="121px"><a style="display:block;margin-left:80px;height:80px;width:100px;" href="http://vk.com/think_positively"target="_blank"

</tr>
<tr>
<td class="index_menu" colspan="2">
<a href="/">Главная</a>
<a href="/?page=marketing">Маркетинг</a>
<a href="/?page=faq">Вопрос-Ответ</a>
<a href="/?page=advert" style="background:none;margin:0;padding:0;">Баннеры</a>
</td></tr>
<tr><td class="index_vklady" colspan="2"></td></tr>
<tr><td class="index_center" colspan="2"></td></tr>
<tr><td class="index_referalu" colspan="2">

</td></tr>
<tr>
<td class="index_stat_left"><font color="#77AF1B"><?php echo $d_users; ?></font> <font color="#FF962D"><?php echo $ut; ?></font></td>
<td class="index_stat_right">
<span class="index_stat_plus">ПОПОЛНИЛИ: <?php echo number_format($d_plus,2,'.',','); ?> РУБ</span>
<span class="index_stat_with">ВЫВЕЛИ: <?php echo number_format($d_with,2,'.',','); ?> РУБ</span>
<span class="index_stat_free">БАЛАНС: <?php echo number_format($free,2,'.',','); ?> РУБ</span>
</td>
</tr>
</table>



<table class="osnova_table" align="center" cellpadding="0px" cellspacing="0px" width="980px">
<tr>
<td class="osnova_left">




<?php if(!USER_LOGGED){ ?>

<?php if(!empty($wrong_lq)){ echo '<div class="wrong_lq">Неверный логин-кошелёк</div>'; } ?>

<div class="osnova_title">Вход в аккаунт</div>

<table class="osnova_vhod" cellpadding="0px" cellspacing="0px">
<tr>
<td colspan="2">
<form id="enter" action="/" method="POST" style="margin:0;padding:0">
<input class="osnova_vhod_input_login" type="text" name="login" placeholder="Логин" maxlength="20">
<input class="osnova_vhod_input_qiwi" type="text" name="qiwi" placeholder="Пароль" maxlength="30">
</form>
</td>
</tr>
<tr>
<td><a class="osnova_vhod_registration" href="/?page=registration">Регистрация</a></td>
<td align="right"><a class="osnova_vhod_enter" href="javascript:with(document.getElementById('enter')){ submit(); }">Войти</a></td>
</tr>
<tr><td colspan="2"><br></td></tr>
</table>

<?php } else { ?>

<div class="osnova_title">Личный кабинет</div>

<div class="cabinet">
<a href="/?page=vklady">Мои вклады</a>
<a href="/?page=popolnit">Пополнить баланс</a>
<a href="/?page=vyvesti">Вывести деньги</a>
<a href="/?page=refs">Мои рефералы</a>
<a href="/logout.php">Выход</a>
</div>

<?php } ?>

<div class="osnova_projects_bottom"></div>

<br>

<div class="osnova_title">Мы принимаем</div>

<div class="osnova_obmen">
<center><a href="https://w.qiwi.com/eggs/main.action" target="_blank"><img src="/images/eggs.png"></a></center>
<center><a href="https://w.qiwi.com/eggs/main.action" target="_blank">QIWI Яйца</a></center>
</div>

<div class="osnova_projects_bottom"></div>

<br>
<div class="osnova_online">На сайте: <font color="#FF8D1C"><?php echo count_online($ip,$time); ?></font></div>



</td>
<td class="osnova_right">

<?php
$nay=0;
if(!empty($_GET['page'])){
$req=$_GET['page'];
$req=str_replace('/?page=','',$req);
if(in_array($req,$inc)){ $nay=1; include ('pages/'.$req.'.php'); }
if(USER_LOGGED && in_array($req,$inc_cab)){ $nay=1; include ('cabinet/'.$req.'.php'); }
if(!USER_LOGGED && $req=='registration'){ $nay=1; include ('pages/registration.php'); }
}


if($nay!=1){ include ('pages/main.php'); }
?>

</td>
</tr>
</table>


<br>

</td>
</tr>
</table>

<table align="center" cellpadding="0px" cellspacing="0px">
<tr><td class="footer"></td></tr>
</table>
