---
title: Blog功能测试
date: 2016-06-20 12:10:00
tags: article
---

做博客的功能测试：代码高亮、数学公式、文章发布等等
<!-- more -->

## 代码高亮测试

### Javascript
```js
import Vue from 'vue'
import Router from 'vue-router'

import * as actions from './../store/actions/user-actions'
import * as config from '../config/config'
import * as util from '../api/util'

Vue.use(Router)
var router = new Router()

export default{
    data () {
        return {
            username: '',
            password: ''
        }
    },
    ready() {
        console.log('ready', this.isLogin)

    },
    created() {
        this.isLogin ? router.go('/home/dashboard') : router.go('/signin')
        console.log('create', this.isLogin)
    },
    vuex: {
        getters: {
            isLogin: ({auth}) => {
                return auth.isLogin
            }
        },
        actions: actions
    }
}
```

### $\rm\TeX$ or $\rm\LaTeX$
```tex
When $a \ne 0$, there are two solutions to \(ax^2 + bx + c = 0\) and they are
$$x = {-b \pm \sqrt{b^2-4ac} \over 2a}.$$

$$\frac{1}{\Bigl(\sqrt{\phi \sqrt{5}}-\phi\Bigr) e^{\frac25 \pi}} = 1+\frac{e^{-2\pi}} {1+\frac{e^{-4\pi}} {1+\frac{e^{-6\pi}} {1+\frac{e^{-8\pi}} {1+\cdots} } } }$$
```

### PHP
```php
<?php

namespace NSC\Model;

class Article extends BaseModel
{
    /**
     * @var string $table 当前主要使用的数据表名
     */
    private $table = 'article_detail';

    /**
     * 删除一篇文章
     *
     * 要先删除关联的消息，比如：ArticleTranslate中的记录
     *
     * @param array $ids 支持同时删除多个记录
     * @return \Doctrine\DBAL\Driver\Statement|int
     */
    public function delete($ids) {
        $translate = new ArticleTranslate();
        $query = $this->conn->createQueryBuilder();

        $translate->delete($ids);

        $stmt = $query->delete($this->table)
            ->where($query->expr()->in('Article_Detail_ID', $ids))
            ->execute();

        return $stmt;
    }
}
```

### HTML
```html
<!DOCTYPE html>
<html lang="en" class="app js no-touch no-android chrome no-firefox no-iemobile no-ie no-ie8 no-ie10 no-ie11 no-ios">
<head>
    <meta charset="utf-8">
    <title>NSC</title>
    <link rel="stylesheet" href="assets/libs/fontawesome/css/font-awesome.css">
    <link rel="stylesheet" href="assets/libs/bootstrap/css/bootstrap.css">
    <link rel="stylesheet" href="assets/css/animate.css">
    <link rel="stylesheet" href="assets/css/font.css">
    <link rel="stylesheet" href="assets/css/icon.css">
    <link rel="stylesheet" href="assets/libs/daterangepicker/daterangepicker.css">
    <link rel="stylesheet" href="assets/libs/icheck/skins/all.css">
    <!--<link rel="stylesheet" href="assets/libs/umeditor/themes/default/css/umeditor.min.css">-->
    <link rel="stylesheet" href="assets/libs/ueditor/themes/default/css/ueditor.min.css">
    <link rel="stylesheet" href="assets/css/app.css">
</head>
<body>
<div id="app"></div>

<script src="assets/libs/jquery/jquery.min.js"></script>
<script src="assets/libs/bootstrap/js/bootstrap.js"></script>
<script src="assets/libs/echarts/echarts.min.js"></script>
<script src="assets/libs/moment/moment-with-locales.min.js"></script>
<script src="assets/libs/daterangepicker/daterangepicker.js"></script>
<script src="assets/libs/icheck/icheck.min.js"></script>
<!--<script src="assets/libs/umeditor/umeditor.config.js"></script>
<script src="assets/libs/umeditor/umeditor.min.js"></script>-->
<script src="assets/libs/ueditor/ueditor.config.js"></script>
<script src="assets/libs/ueditor/ueditor.all.min.js"></script>

<script src="dist/build.js"></script>
</body>
</html>
```

### Java
```java
package org.telegram.mtproto;

import org.telegram.tl.DeserializeException;
import org.telegram.tl.TLContext;
import org.telegram.tl.TLMethod;
import org.telegram.tl.TLObject;

import java.io.IOException;
import java.io.InputStream;

public class HelpGetNearestDc extends TLMethod<NearestDc> {

    private static final int CLASS_ID = 531836966;


    @Override
    public NearestDc deserializeResponse(InputStream stream, TLContext context) throws IOException {
        NearestDc response = (NearestDc) context.deserializeMessage(stream);
        if (response == null) {
            throw new DeserializeException("Unable to deserialize response");
        }
        if (!(response instanceof NearestDc)) {
            throw new DeserializeException("Response has incorrect type");
        }

        System.out.println(response);
        return response;
    }

    @Override
    public int getClassId() {
        return CLASS_ID;
    }

}
```


### SQL
```sql
-- 1. 插入新的权限对象
INSERT INTO OPERATION_OBJECT
(PARENT_OPERATION_OBJECT_ID, OPERATION_OBJECT_CODE, OPERATION_OBJECT_NAME, OPERATION_OBJECT_URL, OPERATION_OBJECT_TYPE)
  SELECT
    OBJECT_ID,
    'findByName' as OPERATION_OBJECT_CODE,
    'Find By Name' as OPERATION_OBJECT_NAME,
    '?r=fetch/findByName' as OPERATION_OBJECT_URL,
    'L' as OPERATION_OBJECT_TYPE
  FROM
    OPERATION_OBJECT
  WHERE
    OPERATION_OBJECT_CODE = 'tools';
COMMIT;

-- 2. 插入权限表（需要先提交上一条记录）
INSERT INTO PRIVILEGE
(ROLE_ID, OPERATION_OBJECT_CODE, OPERATION_OBJECT_ID, CAN_BROWSE, CAN_MODIFY, CAN_ACTION, CAN_DELETE)
  SELECT
    r.ROLE_ID,
    oo.OPERATION_OBJECT_CODE,
    oo.OPERATION_OBJECT_ID,
    1,
    0,
    0,
    0
  FROM
    ROLE r, OPERATION_OBJECT oo
  WHERE
    oo.OPERATION_OBJECT_CODE = 'findByName';
COMMIT;

```


### 数学公式
当$a \ne 0$时，$ax^2 + bx + c = 0$ 有如下两个解：
$$x = {-b \pm \sqrt{b^2-4ac} \over 2a}.$$

复杂数学表达式测试

$$\frac{1}{\Bigl(\sqrt{\phi \sqrt{5}}-\phi\Bigr) e^{\frac25 \pi}} = 1+\frac{e^{-2\pi}} {1+\frac{e^{-4\pi}} {1+\frac{e^{-6\pi}} {1+\frac{e^{-8\pi}} {1+\cdots} } } }$$
