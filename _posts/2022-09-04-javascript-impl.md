---
layout: article
title: '[JavaScript] 應用'
date: 2022-09-04 10:22
tags: JavaScript
---
紀錄開發上常遇到的問題，避免重複踩坑。
<!--more-->
# DataTables動態欄位

參考<https://datatables.net/forums/discussion/60722/dynamic-columns>
```js
$(function() {
    var genColumns = function(data) {
        var columns = [];
        var columnNames = Object.keys(data[0]);
        for (var i in columnNames) {
            columns.push({data: columnNames[i], title: columnNames[i]});
        }
        return columns;
    }
    $.ajax({
        url: "/test",
        success: function (response) {
            $("#dt").DataTable({
                data: response,
                columns: genColumns(response)
            });
        }
    });
});
```