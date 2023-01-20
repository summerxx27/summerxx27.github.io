---
layout: post
title: "ruby实战 获取GithubApi数据导出 Excel"
subtitle: "ruby derive excel"
date: 2023-01-18
author: "summerxx"
header-img: "img/post-bg-2015.jpg"
tags: [ruby]
---

此脚本运行需要申请一个 github api token, 请参照前面几篇文章
此脚本的目的, 就是为了导出一个用户下所有库(repo)详细信息的一个excel表, Demo 地址我放到文章最后
此脚本的使用也很简单如下图

- 安装 spreadsheet , cli-ui, 命令我放在下方
- 运行 main.by
- 输入您要查询的用户名字, 这里可以写自己的 github(summerxx27) 用户名
- 之后选择确认 Yes. No
- 之后文件就会自动导出到桌面

![截屏2023-01-18 15.45.59](https://p.ipic.vip/lkg6qe.png)

![截屏2023-01-18 15.46.23](https://p.ipic.vip/a405yx.png)



使用此脚本钱先 安装必要的gems 包

```
gem install spreadsheet
gem install cli-ui
```

```ruby
require 'uri'
require 'net/http'
require 'net/https'
require 'json'
require 'cli/ui'            # 请输入 xxx的功能
require "spreadsheet/excel" # excel的导出使用

#网络请求类
class Network    
    def Network.request (url, &block)
        uri = URI.parse(url)
        https = Net::HTTP.new(uri.host,uri.port)
        https.use_ssl = true
        req = Net::HTTP::Get.new(uri.path, initheader = {'Content-Type' =>'application/json'})
        req['Authorization'] = 'token token_Id'
        res = https.request(req)
        if res.code == "200"
            data = JSON.parse(res.body)
            # 回调data数据
            yield data
        else 
            puts "请求失败 : Response code = #{res.code}, message = #{res.message}"
        end
    end
end

# Excel 参考 : https://github.com/zdavatz/spreadsheet/blob/master/GUIDE.md
#设置表格的编码为utf-8
Spreadsheet.client_encoding = "utf-8"
#创建表格对象
book = Spreadsheet::Workbook.new
#创建工作表
sheet1 = book.create_worksheet :name => "summerxx专属表"
#格式
sheet1.row(0).height = 28
for i in 0..17
    sheet1.column(i).width = 20
end

#在表格第一行设置分类
sheet1.row(0)[0] = "full_name"
sheet1.row(0)[1] = "html_url"
sheet1.row(0)[2] = "description"
sheet1.row(0)[3] = "forks_count"
sheet1.row(0)[4] = "created_at"
sheet1.row(0)[5] = "updated_at"
sheet1.row(0)[6] = "pushed_at"
sheet1.row(0)[7] = "stargazers_count"
sheet1.row(0)[8] = "subscribers_count"
sheet1.row(0)[9] = "open_issues_count"
sheet1.row(0)[10] = "fork"
sheet1.row(0)[11] = "has_wiki"
sheet1.row(0)[12] = "has_pages"
sheet1.row(0)[13] = "archived"
sheet1.row(0)[14] = "disabled"
sheet1.row(0)[15] = "license"
sheet1.row(0)[16] = "language"

# 参照: https://github.com/Shopify/cli-ui
input = CLI::UI::Prompt.ask('请输入您要查询的用户名: 例如 TRaaSStack:')
url = "https://api.github.com/users/#{input}/repos"
puts "您确定要查询的是这个用户吗: #{input}"
sure = CLI::UI::Prompt.ask('确定请选择 Yes, 否定请选择 No') do |handler|
    handler.option('yes')  { |selection| selection }
    handler.option('no')     { |selection| selection }
end

if sure == "no"
    return
end

Network.request(url) { | data |

    if data 
        #创建一个 names 数组
        names = Array.new
        #遍历得到每个 repo 的对象
        for i in 0..data.count
            repo = data[i]
            # 判断 repo 非空
            if repo
            # puts "full_name = #{repo["full_name"]}, i = #{I}"
            # 添加数据进入数组
            names << repo["full_name"]
            end
        end

        for i in 0..names.count - 1
            name = names[I]
            url = "https://api.github.com/repos/#{name}"
            # puts url
            Network.request(url) { | data |
                # 判断 repo 非空
                if data
                    puts "full_name = #{data["full_name"]}, created_at = #{data["created_at"]}, i = #{I}}"
                    sheet1.row(i + 1)[0] = data["full_name"]
                    sheet1.row(i + 1)[1] = data["html_url"]
                    sheet1.row(i + 1)[2] = data["description"]
                    sheet1.row(i + 1)[3] = data["forks_count"]
                    sheet1.row(i + 1)[4] = data["created_at"]
                    sheet1.row(i + 1)[5] = data["updated_at"]
                    sheet1.row(i + 1)[6] = data["pushed_at"]
                    sheet1.row(i + 1)[7] = data["stargazers_count"]
                    sheet1.row(i + 1)[8] = data["subscribers_count"]
                    sheet1.row(i + 1)[9] = data["open_issues_count"]
                    sheet1.row(i + 1)[10] = data["fork"]
                    sheet1.row(i + 1)[11] = data["has_wiki"]
                    sheet1.row(i + 1)[12] = data["has_pages"]
                    sheet1.row(i + 1)[13] = data["archived"]
                    sheet1.row(i + 1)[14] = data["disabled"]
                    if data["license"] 
                        license = data["license"]
                        name = license["name"]
                        sheet1.row(i + 1)[15] = name
                    end
                    sheet1.row(i + 1)[16] = data["language"]

                        if i == names.count - 1
                            puts "excel 已经导出成功, 请查看桌面"
                            #在指定路径下面创建表格，并写book对象
                            book.write "/Users/summerxx/Desktop/rubyExcel/summerxx专属表.xls"
                        end
                end
            }
        end
    end
}
```

Ruby 环境配置 请看这篇 [https://www.jianshu.com/p/0f5a7a8293f5](https://www.jianshu.com/p/0f5a7a8293f5)

Demo

[https://github.com/summerxx27/XTDeriveGithubDataToExcel](https://github.com/summerxx27/XTDeriveGithubDataToExcel)
