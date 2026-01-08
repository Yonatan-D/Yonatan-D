# 在 Windows 服务器上申请 SSL 证书及自动续签

{docsify-my-updater date:2023-06-06}

?> **win-acme**    
一个简单的 ACME Windows 客户端（基于Let's Encrypt）  
官网：https://www.win-acme.com/  
Github：https://github.com/win-acme/win-acme



## 背景

在 Windows 服务器上定时续期 SSL 证书，把证书文件放到可被下载的目录供其它服务器更新使用。WinACME 提供了多种签证方式，并且能添加到计划任务自动续签，本次我们使用的是阿里云的 DNS API。



## 下载地址

[win-acme v2.2.5.1541](https://github.com/win-acme/win-acme/releases/download/v2.2.5.1541/win-acme.v2.2.5.1541.x64.pluggable.zip)



## 自定义脚本

需要用到两个脚本：

- dnsapi_ali.ps1 - 阿里云 DNS 自动验证（脚本放在文末）
- release.ps1 - 发布后执行流程 （按你需求更改）

修改 `dnsapi_ali.ps1` 里的第 7, 8 行，将 AliKeyId, AliSecretInsecure 改为你的：

```powershell
param(
	[string]$Task,
	[string]$DomainName,
	[string]$RecordName,
	[string]$TxtValue
)
$AliKeyId = '阿里云 AccessKey ID'
$AliSecretInsecure = '阿里云 AccessKey Secret'
```

利用 `release.ps1` 脚本把生成的证书复制到别处并重命名：

```powershell
copy-item .\exportssl\_.yonatan.cn-chain.pem .\SSL\fullchain.crt
copy-item .\exportssl\_.yonatan.cn-key.pem .\SSL\private.pem
copy-item .\exportssl\_.yonatan.cn.pfx .\SSL\certificate.pfx
```



## 运行 win-acme

运行 wacs.exe ，操作为：

- **M** (Create certificate (full options))
- 选择 **2** (Manual input)
- A host name to get a certificate for. This may be a comma-separated list. Host: ***.yonatan.cn (输入你的域名)**
- Friendly name '[Manual] 刚刚输入的域名'. <Enter> to accept or type desired name: **<直接回车>**
- 选择 **8** ([dns-01] Create verification records with your own script)
- Path to script that creates the validation TXT record.  DnsCreateScript: **.\dnsapi_ali.ps1 (阿里云 DNS 自动验证脚本)**
- How to delete records after validation: **3 (Do not delete) ** 
- DnsCreateScriptArguments: **<直接回车>**
- What kind of private key should be used for the certificate?: **2 (RSA key)**
- ==== 这里开始是逐一添加证书导出格式，先导出PEM ====
- 选择 **2** (PEM encoded files (Apache, nginx, etc.))
- .pem files are exported to this folder.  File path: **.\exportssl (导出的目录)** 
- Password to set for the private key .pem file. Choose from the menu: **1** None **(不设密码)**
- ==== Would you like to store it in another way too? ====
- ==== 又重复回上述菜单项，继续添加PFX ====
- 选择 **3** (PFX archive)
- Path to write the .pfx file to. File path: **.\exportssl (导出的目录)**
- Password to set for .pfx files exported to the folder. Choose from the menu: **2** (Type/paste in console) **这里选择设置 pfx 导入密码（东方通：转成 jks 需要密码）**
- PfxPassword: **yonatan (输入你的密码)**
- Save to vault for future reuse? (y/n*)  **y (保存密码以便下次快速使用)** 
- Please provide a unique name to reference this secret: **yont (给密码取个唯一名称)** 
- ==== 现在可以选择 **5** (No (additional) store steps)  退出了 ====
- Which installation step should run first?: **2: Start external script or program** (选择导出后执行的脚本，没有则选3)
- 输入脚本路径： **.\release.ps1** ，直接回车（不用输入Parameters）
- Add another installation step?: 选择 **3** No (additional) installation steps **没有要执行的脚本了**
- 中间步骤都是选择 **y**
- **此时，证书已经生成到 exportssl 目录，并且 ssl 目录里就是我们最终想要的**
- Do you want to specify the user the task will run as? (y/n*) - **yes 是否要指定任务将以哪个用户的身份运行？** 
- Enter the username (Domain\username): administrator **输入windows 用户名**
- Enter the user's password: \*\*\*\*\*\*\*  **输入windows 用户密码**



## 实际效果

exportssl 目录下能看到生成的证书文件

任务计划程序可以看到 win-acme 在每天的 09:00 执行检查证书的任务计划



## PS

### dnsapi_ali.ps1

```powershell
param(
	[string]$Task,
	[string]$DomainName,
	[string]$RecordName,
	[string]$TxtValue
)
$AliKeyId = '阿里云 AccessKey ID'
$AliSecretInsecure = '阿里云 AccessKey Secret'


$script:WellKnownDirs = @{
    LE_PROD = 'https://acme-v02.api.letsencrypt.org/directory';
    LE_STAGE = 'https://acme-staging-v02.api.letsencrypt.org/directory';
}
$script:HEADER_NONCE = 'Replay-Nonce'
$script:USER_AGENT = "iEdon-Modified; PowerShell/$($PSVersionTable.PSVersion)"
$script:COMMON_HEADERS = @{'Accept-Language'='en-us,en;q=0.5'}


$script:UseBasic = @{}
if ('UseBasicParsing' -in (Get-Command Invoke-WebRequest).Parameters.Keys) {
    $script:UseBasic.UseBasicParsing = $true
}

function Get-DateTimeOffsetNow {
    [CmdletBinding()]
    param()
    [System.DateTimeOffset]::Now
}
function Add-DnsTxtAliyun {
    [CmdletBinding(DefaultParameterSetName='Insecure')]
    param(
        [Parameter(Mandatory,Position=0)]
        [string]$RecordName,
        [Parameter(Mandatory,Position=1)]
        [string]$TxtValue,
        [Parameter(Mandatory,Position=2)]
        [string]$AliKeyId,
        [Parameter(ParameterSetName='Secure',Mandatory,Position=3)]
        [securestring]$AliSecret,
        [Parameter(ParameterSetName='Insecure',Mandatory,Position=3)]
        [string]$AliSecretInsecure,
        [Parameter(ValueFromRemainingArguments)]
        $ExtraParams
    )

    if ('Insecure' -eq $PSCmdlet.ParameterSetName) {
        $AliSecret = ConvertTo-SecureString $AliSecretInsecure -AsPlainText -Force
    }

    try { $zoneName = Find-AliZone $RecordName $AliKeyId $AliSecret } catch { throw }
    Write-Debug "Found zone $zoneName"
	
    $recShort = $RecordName -ireplace [regex]::Escape(".$zoneName"), [string]::Empty
    if ($recShort -eq $RecordName) { $recShort = '@' }

    try {
        $queryParams = "DomainName=$zoneName","RRKeyWord=$recShort","ValueKeyWord=$TxtValue",'TypeKeyWord=TXT'
        $response = Invoke-AliRest DescribeDomainRecords $queryParams $AliKeyId $AliSecret
    } catch { throw }

    if ($response.TotalCount -gt 0) {
        Write-Debug "Record $RecordName already contains $TxtValue. Nothing to do."
    } else {
        Write-Verbose "Adding a TXT record for $RecordName with value $TxtValue"
        $queryParams = "DomainName=$zoneName","RR=$recShort","Value=$TxtValue",'Type=TXT'
        Invoke-AliRest AddDomainRecord $queryParams $AliKeyId $AliSecret | Out-Null
    }
}

function Remove-DnsTxtAliyun {
    [CmdletBinding(DefaultParameterSetName='Insecure')]
    param(
        [Parameter(Mandatory,Position=0)]
        [string]$RecordName,
        [Parameter(Mandatory,Position=1)]
        [string]$TxtValue,
        [Parameter(Mandatory,Position=2)]
        [string]$AliKeyId,
        [Parameter(ParameterSetName='Secure',Mandatory,Position=3)]
        [securestring]$AliSecret,
        [Parameter(ParameterSetName='Insecure',Mandatory,Position=3)]
        [string]$AliSecretInsecure,
        [Parameter(ValueFromRemainingArguments)]
        $ExtraParams
    )

    if ('Insecure' -eq $PSCmdlet.ParameterSetName) {
        $AliSecret = ConvertTo-SecureString $AliSecretInsecure -AsPlainText -Force
    }

    try { $zoneName = Find-AliZone $RecordName $AliKeyId $AliSecret } catch { throw }
    Write-Debug "Found zone $zoneName"

    $recShort = $RecordName -ireplace [regex]::Escape(".$zoneName"), [string]::Empty
    if ($recShort -eq $RecordName) { $recShort = '@' }

    try {
        $queryParams = "DomainName=$zoneName","RRKeyWord=$recShort","ValueKeyWord=$TxtValue",'TypeKeyWord=TXT'
        $response = Invoke-AliRest DescribeDomainRecords $queryParams $AliKeyId $AliSecret
    } catch { throw }

    if ($response.TotalCount -gt 0) {
        Write-Verbose "Removing TXT record for $RecordName with value $TxtValue"
        $id = $response.DomainRecords.Record[0].RecordId
        Invoke-AliRest DeleteDomainRecord @("RecordId=$id") $AliKeyId $AliSecret | Out-Null
    } else {
        Write-Debug "Record $RecordName with value $TxtValue doesn't exist. Nothing to do."
    }
}

function Save-DnsTxtAliyun {
    [CmdletBinding()]
    param(
        [Parameter(ValueFromRemainingArguments)]
        $ExtraParams
    )
}

function Invoke-AliRest {
    [CmdletBinding()]
    param(
        [Parameter(Mandatory,Position=0)]
        [string]$Action,
        [Parameter(Position=1)]
        [string[]]$ActionParams,
        [Parameter(Mandatory,Position=2)]
        [string]$AccessKeyId,
        [Parameter(Mandatory,Position=3)]
        [securestring]$AccessSecret
    )

    $apiBase = 'https://alidns.aliyuncs.com'
    $allParams = $ActionParams + @(
        "AccessKeyId=$AccessKeyId",
        "Action=$Action",
        'Format=json',
        'SignatureMethod=HMAC-SHA1',
        "SignatureNonce=$((New-Guid).ToString())",
        'SignatureVersion=1.0',
        "Timestamp=$((Get-DateTimeOffsetNow).UtcDateTime.ToString('yyyy-MM-ddTHH\%3Amm\%3AssZ'))",
        "Version=2015-01-09"
    ) | Sort-Object

    $strToSign = [uri]::EscapeDataString($allParams -join '&')
    $strToSign = "GET&%2F&$strToSign"
    Write-Debug $strToSign
    $stsBytes = [Text.Encoding]::UTF8.GetBytes($strToSign)

    $secPlain = (New-Object PSCredential "user",$AccessSecret).GetNetworkCredential().Password
    $secBytes = [Text.Encoding]::UTF8.GetBytes("$secPlain&")
    $hmac = New-Object Security.Cryptography.HMACSHA1($secBytes,$true)
    $sig = [Convert]::ToBase64String($hmac.ComputeHash($stsBytes))
    $sigUrl = [uri]::EscapeDataString($sig)
    Write-Debug $sig

    $uri = "$apiBase/?$($allParams -join '&')&Signature=$sigUrl"
    Invoke-RestMethod $uri @script:UseBasic -EA Stop
}

function Find-AliZone {
    [CmdletBinding()]
    param(
        [Parameter(Mandatory,Position=0)]
        [string]$RecordName,
        [Parameter(Mandatory,Position=1)]
        [string]$AliKeyId,
        [Parameter(Mandatory,Position=2)]
        [securestring]$AliSecret
    )

    if (!$script:AliRecordZones) { $script:AliRecordZones = @{} }

    if ($script:AliRecordZones.ContainsKey($RecordName)) {
        return $script:AliRecordZones.$RecordName
    }

    $pieces = $RecordName.Split('.')
    for ($i=1; $i -lt ($pieces.Count-1); $i++) {
        $zoneTest = "$( $pieces[$i..($pieces.Count-1)] -join '.' )"
        Write-Debug "Checking $zoneTest"
        try {
            $response = Invoke-AliRest DescribeDomains @("KeyWord=$zoneTest") $AliKeyId $AliSecret
            if ($response.TotalCount -gt 0) {
                $script:AliRecordZones.$RecordName = $response.Domains.Domain[0].DomainName # or PunyCode?
                return $script:AliRecordZones.$RecordName
            }
        } catch { throw }
    }

    throw "No zone found for $RecordName"
}
if ($Task -eq 'create'){
	Add-DnsTxtAliyun $RecordName $TxtValue $AliKeyId $AliSecretInsecure
}
if ($Task -eq 'delete'){
	Remove-DnsTxtAliyun $RecordName $TxtValue $AliKeyId $AliSecretInsecure
}
```

