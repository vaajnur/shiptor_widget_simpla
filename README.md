1. AddPackage.php , composer.json, shiptor.js , vendor скопировать с корень сайта. Или все это без папки vendor, далее просто запустить composer install. vendor создасться автоматом.   

2. email_order.tpl - скопировать в /design/amourshop/html/

3. email_order_admin.tpl - скопировать в /simpla/design/amourshop/html/

4. /design/amourshop/html/cart.tpl - в начале добавть следующ. код




5. В админке добавить Способ доставки Shiptor. Остальные убрать. Для Shiptor добавить Способы оплаты.

6. в папку конфиг закинуть shiptor_api_key.txt с ключом апи 
 ( API токен для интеграции https://api.shiptor.ru/shipping/v1 )

7. view/CartView.php

в блок   проверки нажатия формы
if(isset($_POST['checkout']) && !empty($_POST['shiptor']))
    {

        $Shiptor          = json_decode($this->request->post('shiptor'), true);

        // $order->address = '';
        // $captcha_code =  

в конце этого блока, до отправки сообщений

                   if(isset($Shiptor['pvz'])){
                        //array_reverse(sSiptor['pvz']['address']);
                        $this->orders->update_order($order->id, array(
                                'delivery_price' => str_replace(' руб.' , '' , $Shiptor['pvz']['cost']), 
                                //'separate_delivery'=>$delivery->separate_payment, 
                                'address' =>  $Shiptor['pvz']['address']
                            )
                        );
                    } elseif(isset($Shiptor['courier'])){                        
                        //array_reverse(sSiptor['contacts']['address']);
                        $this->orders->update_order($order->id, array(
                                'delivery_price' => $Shiptor['courier']['cost']['total']['sum'], 
                                //'separate_delivery'=>$delivery->separate_payment,                                 
                                'address' => $Shiptor['location']['city'] . ' , ' . $Shiptor['street'] . ' , ' . $Shiptor['dom']
                            )
                        );                        
                    }  elseif(isset($Shiptor['pochta'])){  
                        
                            $this->orders->update_order($order->id, array(
                                'delivery_price' => $Shiptor['pochta']['cost']['total']['sum'], 
                                //'separate_delivery'=>$delivery->separate_payment,                                 
                                'address' => $Shiptor['location']['city'] . ' , ' . $Shiptor['street'] . ' , ' . $Shiptor['dom']
                            )
                        );  
                    }

8. в api/Cart.php вывести length, width , weight 

 /////////////////////////////////////////
						$product_features = $this->features->get_product_options($item->variant->product_id);
						foreach($product_features as $feature)	{
                                                        if($feature->name == 'Ширина' && $feature->value){
								$width = $feature->value;
							} 
                                                        // 'Длина, см'
                                                        if($feature->feature_id== 184 && $feature->value){
								$length = $feature->value;
                                                        }
                                                        if($feature->name == 'Вес' && $feature->value){
								$weight = $feature->value;
							}
                                                        
						}

                                                $purchase->width = !empty($width) ? $width : 0;
                                                $purchase->length = !empty($length) ? $length : 0;
                                                $purchase->weight = !empty($weight) ? $weight : 0;
