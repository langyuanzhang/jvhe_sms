# jvhe_sms
聚合短信

1.	在浏览器中搜索聚合数据官网或输入网址https://www.juhe.cn/

2.	点击官网首页右上角的登录注册按钮，先使用客户的资料注册账号，再登录。（记得整理记录账号密码）

3.	登陆后点击个人中心，进入后台操作页面。

4.	点击“未认证”进行企业认证（图为已认证的账号），按照步骤填写信息，注意要用新的三证合一的社会编码。
 
5.	在“我的数据”=》全部数据=》点击“申请新数据”按钮，

6.	选择“即时通讯”=》短信API服务=》点击“立即申请”按钮
 
7.	在“我的数据”中查看申请情况，一般十几分钟或者几个小时就能申请成功。

    认证成功后，点击“模板”按钮
 
8.	进入页面后，在短信模板点击下方链接，第一次需要先点击链接进入打款认证。
 
9.	在该页面下，填写手机号码，并生成专属汇款账号。让客户尽快向该账号汇款

10.	返回“我的数据”，在已认证的短信API服务栏后，点击“模板”按钮，进行模板内容等信息的填写，最后提交审核。

11.	审核通过后在该界面下会有模板ID，需要整理交给开发者

12.	如果需要购买十条以上需要去购买服务


```
//调用方法 
$sms_res = $this->sendSms($tradeNo,$paymoney);

//聚合数据短信通知接口
public function sendSms($tradeNo,$paymoney)
{
    $od = OrderGoods::where('order_id','eq',$tradeNo)->find();

    //订单中间表
    $md = Db::name('order_goods_middle')->where('order_id','eq',$tradeNo)->select();

    $shopall='';//所有商品
    foreach ($md as $km => $vm) {
        $shopall= $shopall.$vm['goods_name'].'*'.$vm['number'].'、';
    }

    $order_id=$tradeNo;//订单号
    $amount=$paymoney;//订单总额
    $mobile=$od['phone'];//收货联系
    $realname=$od['realname'];//收货人
    $address = $od['address'];//收货地址
    $goods=$shopall;//商品内容
    $create_time=$od['create_time'];//下单时间

    // 订单发货通知，订单号码#order_id#，订单总额#amount#，收货联系#mobile#，收货人#realname#，收货地址#address#，商品内容#goods#,发货时间#create_time#
    $tpl_value="#order_id#=:".$tradeNo."&#amount#=:".$amount."&#mobile#=:".$mobile."&#realname#=:".$realname."&#address#=:".$address."&#goods#=:".$goods."&#create_time#=:".$create_time;

    $smsnot = Db::name('sms_notification')->where('status','eq',1)->find();

    if(empty($smsnot)){
        return json_encode(array('status'=>0,'message'=>'没有开启短信通知'));
    }
    $mobile=$smsnot['phone'];//管理员电话
    $access_key=$smsnot['access_key'];//申请的APPKEY
    $access_key_secret=$smsnot['access_key_secret'];//您申请的短信模板ID，根据实际情况修改
    $sendUrl = 'http://v.juhe.cn/sms/send'; //短信接口的URL
    $smsConf = array(
        'key'   => $access_key, //您申请的APPKEY
        'mobile'    => $mobile, //接受短信的用户手机号码
        'tpl_id'    => $access_key_secret, //您申请的短信模板ID，根据实际情况修改
        'tpl_value' => urlencode($tpl_value),        //'#code#=1234&#company#=聚合数据' //您设置的模板变量，根据实际情况修改
    );
    
    $content = juhecurl($sendUrl,$smsConf,1); //请求发送短信
    
    if($content){
        $result = json_decode($content,true);
        $error_code = $result['error_code'];
        if($error_code == 0){
            //状态为0，说明短信发送成功
            return json_encode("短信发送成功,短信ID：".$result['result']['sid']);
        }else{
            //状态非0，说明失败
            $msg = $result['reason'];
            return json_encode("短信发送失败(".$error_code.")：".$msg);
        }
    }else{
        //返回内容异常，以下可根据业务逻辑自行修改
        return json_encode("请求发送短信失败");
    }
}


//放在common.php
/**
 * 请求接口返回内容
 * @param  string $url [请求的URL地址]
 * @param  string $params [请求的参数]
 * @param  int $ipost [是否采用POST形式]
 * @return  string
 */
function juhecurl($url,$params=false,$ispost=0){
    $httpInfo = array();
    $ch = curl_init();
    curl_setopt( $ch, CURLOPT_HTTP_VERSION , CURL_HTTP_VERSION_1_1 );
    curl_setopt( $ch, CURLOPT_USERAGENT , 'Mozilla/5.0 (Windows NT 5.1) AppleWebKit/537.22 (KHTML, likeGecko) Chrome/25.0.1364.172 Safari/537.22' );
    curl_setopt( $ch, CURLOPT_CONNECTTIMEOUT , 30 );
    curl_setopt( $ch, CURLOPT_TIMEOUT , 30);
    curl_setopt( $ch, CURLOPT_RETURNTRANSFER , true );
    if( $ispost )
    {
        curl_setopt( $ch , CURLOPT_POST , true );
        curl_setopt( $ch , CURLOPT_POSTFIELDS , $params );
        curl_setopt( $ch , CURLOPT_URL , $url );
    }
    else
    {
        if($params){
            curl_setopt( $ch , CURLOPT_URL , $url.'?'.$params );
        }else{
            curl_setopt( $ch , CURLOPT_URL , $url);
        }
    }
    $response = curl_exec( $ch );
    if ($response === FALSE) {
        //echo "cURL Error: " . curl_error($ch);
        return false;
    }
    $httpCode = curl_getinfo( $ch , CURLINFO_HTTP_CODE );
    $httpInfo = array_merge( $httpInfo , curl_getinfo( $ch ) );
    curl_close( $ch );
    return $response;
}
```
