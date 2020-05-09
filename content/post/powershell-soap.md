---
title: "PowerShell SOAP"
date: 2013-11-18
draft: false
---

Пример вызова метода Web сервиса посредством `SOAP`:

``` powershell
$secpasswd = ConvertTo-SecureString "password" -AsPlainText -Force
$mycreds = New-Object System.Management.Automation.PSCredential ("userName", $secpasswd)

$uri = "http://localhost/IB/ws/example?wsdl"

try {
    $service = New-WebServiceProxy -Uri $uri -Credential $mycreds
} catch {
    Write-Error $_ -ErrorAction:'SilentlyContinue'
}

if($service -ne $null){
    try{
        $version = $service.GetVersion()
        echo $version
    }
    catch{
        Write-Error $_ -ErrorAction:'SilentlyContinue'
    }
}
```

В данном случае использовался вызов Web сервиса `1C:Предприятие 8.2`.
