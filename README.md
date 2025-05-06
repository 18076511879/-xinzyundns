# 新云二级域名分发系统

![PHP Version](https://img.shields.io/badge/PHP-7.4%2B-blueviolet)
![MySQL Version](https://img.shields.io/badge/MySQL-5.7%2B-orange)
![License](https://img.shields.io/badge/License-MIT-green)

**一个专业的二级域名分发管理系统**，支持多DNS服务商API集成，提供完整的域名生命周期管理解决方案。

![系统界面预览](screenshots/dashboard.png)

## 目录

- [功能特性](#功能特性)
- [系统架构](#系统架构)
- [环境要求](#环境要求)
- [安装指南](#安装指南)
- [配置说明](#配置说明)
- [使用文档](#使用文档)
- [API集成](#api集成)
- [安全措施](#安全措施)
- [扩展开发](#扩展开发)
- [贡献指南](#贡献指南)
- [许可证](#许可证)

## 功能特性

### 核心功能矩阵

| 模块         | 功能描述                                                                 |
|--------------|--------------------------------------------------------------------------|
| 域名管理      | 支持A/CNAME/MX/TXT记录类型，实时同步DNS解析                             |
| 多厂商支持    | 已集成Cloudflare/Aliyun/DNSPod，支持扩展其他DNS服务商                   |
| 用户系统      | 分级权限控制（管理员/普通用户），支持密码加密存储                       |
| 安全审计      | 操作日志记录，登录失败锁定机制                                         |
| API接口      | 提供RESTful API进行域名操作                                             |
| 统计报表      | 可视化域名分布统计，操作频率监控                                       |

### 技术亮点
- **现代架构**：采用MVC模式开发，代码解耦度高
- **高性能设计**：数据库查询优化，支持批量操作
- **响应式布局**：完美适配PC/平板/手机端
- **开放扩展**：模块化设计，方便集成新DNS服务商

## 系统架构

### 目录结构
```
domain-system/
├── app/                 # 应用核心
│   ├── Controllers/     # 控制器
│   ├── Models/          # 数据模型
│   ├── Libraries/       # 第三方库
│   └── Config/          # 配置文件
├── public/              # 公共资源
│   ├── assets/          # 静态文件
│   └── index.php        # 入口文件
├── views/               # 视图模板
├── install/             # 安装程序
└── tests/               # 单元测试
```

### 技术栈
| 组件         | 技术选型                     |
|--------------|-----------------------------|
| 前端框架     | Bootstrap 5 + jQuery        |
| 后端语言     | PHP 7.4+                    |
| 数据库       | MySQL 5.7+/MariaDB 10.3+    |
| API通信      | cURL + Guzzle               |
| 安全机制     | CSRF保护 + 输入验证         |

## 环境要求

### 服务器要求
- PHP 7.4+（需启用扩展：mysqli, curl, mbstring）
- MySQL 5.7+ 或 MariaDB 10.3+
- Web服务器（推荐Nginx）
- 50MB以上磁盘空间

### 推荐配置
```ini
; php.ini 优化建议
memory_limit = 128M
max_execution_time = 30
post_max_size = 16M
upload_max_filesize = 16M
```

## 安装指南

### 快速安装步骤
1. 上传文件到服务器
2. 设置目录权限：
   ```bash
   chmod -R 755 storage/
   chmod 755 public/assets/
   ```
3. 访问 `http://yourdomain.com/install`
4. 完成数据库配置
5. 删除install目录

### 伪静态配置（Nginx示例）
```nginx
server {
    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }
    
    location ~ /\.ht {
        deny all;
    }
}
```

## 配置说明

### 关键配置文件
1. `app/Config/database.php` - 数据库连接设置
2. `app/Config/dns_providers.php` - DNS服务商密钥配置
3. `app/Config/routes.php` - 自定义路由规则

### DNS服务商配置示例
```php
// Cloudflare配置
'cloudflare' => [
    'api_key' => 'your_global_api_key',
    'email' => 'account@example.com',
    'zone_id' => 'your_zone_id'
],

// 阿里云配置
'alidns' => [
    'access_key' => 'your_access_key',
    'access_secret' => 'your_secret_key',
    'region' => 'cn-hangzhou'
],
```

## 使用文档

### 管理员操作流程
1. 登录后台 `http://domain.com/admin`
2. 添加DNS服务商凭证
3. 创建主域名记录
4. 审核用户提交的二级域名

### 开发者API
```http
POST /api/v1/domains
Content-Type: application/json
Authorization: Bearer {token}

{
    "domain": "example.com",
    "subdomain": "shop",
    "type": "CNAME",
    "target": "cdn.example.net"
}
```

## API集成

### 支持的服务商
| 服务商       | 支持功能                  | 官方文档链接                          |
|--------------|--------------------------|---------------------------------------|
| Cloudflare   | 全功能支持                | https://api.cloudflare.com/           |
| 阿里云DNS     | 除DNSSEC外全部功能        | https://help.aliyun.com/product/29697.html |
| DNSPod       | 基础解析功能              | https://docs.dnspod.cn/api/           |

## 安全措施

### 安全机制清单
- 数据库密码加密存储（bcrypt算法）
- 输入参数过滤（白名单验证）
- 会话固定保护
- 密码复杂度策略：
  - 最少8个字符
  - 必须包含大写字母和数字
  - 禁止使用常见弱密码

### 安全审计建议
```bash
# 定期检查日志
tail -f /var/log/nginx/access.log
grep 'POST /login' access.log
```

## 扩展开发

### 添加新DNS服务商
1. 在`Libraries/`创建新类继承DNS接口
2. 实现核心方法：
   ```php
   interface DNSProvider {
       public function addRecord($domain, $type, $name, $content);
       public function deleteRecord($recordId);
       public function updateRecord($recordId, $newData);
   }
   ```
3. 在`DNSFactory`注册新服务商

### 扩展建议
- 集成Let's Encrypt自动SSL签发
- 添加域名到期提醒功能
- 实现多语言支持

## 贡献指南

欢迎通过以下方式参与贡献：
1. 提交Pull Request
2. 报告Issues
3. 完善文档
4. 翻译多语言版本

代码规范：
- 遵循PSR-12编码规范
- 重要方法必须包含PHPDoc注释
- 新增功能需附带单元测试

## 许可证

本项目采用 MIT 许可证开放源代码

```text
 The MIT License (MIT)
Copyright © 2025 <copyright holders>

Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated documentation files (the “Software”), to deal in the Software without restriction, including without limitation the rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the Software, and to permit persons to whom the Software is furnished to do so, subject to the following conditions:

The above copyright noti# -xinzyundns
新云二级域名分发系统xinzyundns
