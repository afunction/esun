### 玉山銀行虛擬帳號 generator / payment callback parser


##### 注意事項 (玉山金流地雷)

1. 玉山 callback server 有 DNS Cache，如果 production 換 IP 網址沒換，一樣會送到到舊的 IP
2. callback action 如果沒有回傳 `render_esun_ok` 就會連續 query 10 次

##### 安裝

`gem 'esun'`

`bundle install`


```ruby
# config/initializer/esun.rb
::Esun::ATM.company_code = Setting.esun_company_code # settinglogic way
# or
::Esun::ATM.company_code = '92837'
```


##### 產生繳費代碼

```
order_id = 10
amount = 1000
expire = Date.today + 3.days
::Esun::ATM.build_vaccount(order_id, amount, expire)
```

##### payment callbacks

```ruby
# config/router.rb
post "payment/esun"
```


```ruby
# app/controllers/payment_controller.rb
class PaymentController < ActionController::Base
  include ::Esun::CallbackHelper
  add_allow_ip '192.168.3.10' # 增加玉山 server 以外的 ip to white list
  set_esun_callback_action :esun # 指定 esun callback action

  # POST /payment/esun
  def esun
    payment_params.data       # 原始 post 過來的資料

    payment_params.oid         # 訂單編號
    payment_params.amount      # 金額
    payment_params.pay_time    # 付款時間
    payment_params.handle_date # 忘了是什麼?! 知道的人告訴我一下

    # .... 你的 business logic

    # 回應 200 - OK
    render_esun_ok
  end
end
```


##### 有任何問題

請 email 至 eddie [at] visionbundles.com
