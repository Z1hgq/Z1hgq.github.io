---
layout:     post
title:      "js利用文件头判断文件类型"
subtitle:   "校验文件真实类型"
date:       2019-12-10 18:40:25
author:     "Z1hgq"
header-img: "img/post-bg-alitrip.jpg"
catalog: true
tags:
    - JavaScript
---

> “推荐一本小说--《间客》”

---
### 校验函数
```js
    // 校验上传文件真实类型
    // file 是blob对象
    // typeArray是校验的文件类型的文件头数组
    getFileMimeType: (file, typeArray) => {
        const reader = new FileReader();
        reader.readAsArrayBuffer(file);
        return new Promise((resolve, reject) => {
            reader.onload = (event) => {
                try {
                    let buffer = [...Buffer.from(event.target.result)];
                    // 仅要文件的前四位就够了
                    buffer = buffer.splice(0, 4);
                    buffer.forEach((num, i, arr) => {
                        arr[i] = num.toString(16).padStart(2, '0');
                    });
                    let lowerCaseTypeArray = [];
                    for (let item of typeArray) {
                        item = item.toLowerCase();
                        lowerCaseTypeArray.push(item);
                    }
                    resolve(lowerCaseTypeArray.includes(buffer.join('').toLowerCase));
                } catch (e) {
                    // 读取文件头出错 默认不是合法文件类型
                    reject();
                }
            };
        });
    }
```
---
### antd beforeUpload阻止上传

使用upload上传图片，如果beforeUpload直接返回false，仍然会执行onChange，图片会继续上传。
这里是采用在beforeUpload函数中返回一个promise来解决的，这样如果返回false就不会再继续上传了。
官网说明:返回一个 Promise 对象，Promise 对象 reject 时则停止上传，resolve 时开始上传
```js
beforeUpload = file => {
    return new Promise(async (resolve, reject) => {
        let isXlsx = /.xlsx|.xls$/.test(file.name);
        this.isExcel = await commonFun.getFileMimeType(file, ["504b0304", "d0cf11e0"]);
        const overSize = file.size / 1024 / 1024 > 10;  
        if (!isXlsx || !this.isExcel) {
            message.warn(i18next.t('请上传.xlsx或.xls文件'));
            return reject(false);
        }
        if (overSize) {
            message.warn(i18next.t('文件大小不能超过10M'));
            return reject(false);
        }
        return resolve(true);
    });
};
```
---
### 常见文件格式文件头

部分文件头或许有偏差,并且我在项目中遇见过两张不同的原生jpg图片，取4位文件头，一个是ffd8ff，另一个是ffd8ffe1，因此在图片检验的时候可适当调整取的文件头的位数和加一些特殊的校验。

```java
    /** JPEG  (jpg)*/
    JPEG("JPG", "FFD8FF"),
 
    /** PNG */
    PNG("PNG", "89504E47"),
 
    /** GIF */
    GIF("GIF", "47494638"),
  
    /** TIFF (tif)  */
    TIFF("TIF", "49492A00"),
 
    /** Windows bitmap (bmp) */
    BMP("BMP","424D"),
    
    BMP_16("BMP","424D228C010000000000"), //16色位图(bmp) 
    
    BMP_24("BMP","424D8240090000000000"), //24位位图(bmp)  
    
    BMP_256("BMP","424D8E1B030000000000"), //256色位图(bmp)     
 
    /** CAD  (dwg) */
    DWG("DWG", "41433130"),
 
    /** Adobe photoshop  (psd)*/
    PSD("PSD", "38425053"),
 
    /** Rich Text Format  (rtf)*/
    RTF("RTF", "7B5C727466"),
 
    /** XML */
    XML("XML", "3C3F786D6C"),
 
    /** HTML (html)*/
    HTML("HTML", "68746D6C3E"),
    
    /** Email [thorough only] (eml)*/
    EML("EML", "44656C69766572792D646174653A"),  
    
    /** Outlook Express (dbx) */
    DBX("DBX", "CFAD12FEC5FD746F "),
 
    /** Outlook (pst)*/
    PST("", "2142444E"),
 
    /** doc;xls;dot;ppt;xla;ppa;pps;pot;msi;sdw;db */
    OLE2("OLE2", "0xD0CF11E0A1B11AE1"),
 
    /** Microsoft Word/Excel 注意：word 和 excel的文件头一样 */
    XLS("XLS", "D0CF11E0"),
    
    /** Microsoft Word/Excel 注意：word 和 excel的文件头一样 */
    DOC("DOC", "D0CF11E0"),
    
    /** Microsoft Word/Excel 2007以上版本文件 注意：word 和 excel的文件头一样 */
    DOCX("DOCX", "504B0304"),  
 
    /** Microsoft Word/Excel 2007以上版本文件 注意：word 和 excel的文件头一样 504B030414000600080000002100*/
    XLSX("XLSX", "504B0304"), 
 
    /** Microsoft Access (mdb)*/
    MDB("MDB", "5374616E64617264204A"),
 
    /** Word Perfect (wpd)*/
    WPB("WPB", "FF575043"),
 
    /** Postscript */
    EPS("EPS", "252150532D41646F6265"),
 
    /** Postscript */
    PS("PS", "252150532D41646F6265"),
 
    /** Adobe Acrobat (pdf)  */
    PDF("PDF", "255044462D312E"),
    
    /** Quicken (qdf) */
    QDF("qdf", "AC9EBD8F"),
    
    /** QuickBooks Backup (qdb) */
    QDB("qbb", "458600000600"),
    
    /** Windows Password  (pwl)*/
    PWL("PWL", "E3828596"),
 
    /** ZIP Archive */
    ZIP("", "504B0304"),
 
    /** ARAR Archive */
    RAR("", "52617221"),
 
    /** WAVE (wav) */
    WAV("WAV", "57415645"),
 
    /** AVI */
    AVI("AVI", "41564920"),
 
    /** Real Audio (ram)*/
    RAM("RAM", "2E7261FD"),
 
    /** Real Media (rm) rmvb/rm相同  */
    RM("RM", "2E524D46"),
    
    /** Real Media (rm) rmvb/rm相同  */
    RMVB("RMVB", "2E524D46000000120001"),  
    
    /** MPEG (mpg)  */
    MPG("MPG", "000001BA"),
    
    /** Quicktime  (mov)*/
    MOV("MOV", "6D6F6F76"),
 
    /** Windows Media (asf) */
    ASF("ASF", "3026B2758E66CF11"),
 
    /** ARJ Archive */
    ARJ("ARJ", "60EA"),
 
    /** MIDI (mid) */
    MID("MID", "4D546864"),
    
    /** MP4 */  
    MP4("MP4", "00000020667479706D70"), 
    
    /** MP3 */  
    MP3("MP3", "49443303000000002176"),  
    
    /** FLV */  
    FLV("FLV", "464C5601050000000900"), 
    
    /** 1F8B0800000000000000 */
    GZ("GZ", "1F8B08"),
    
    /** CSS */ 
    CSS("CSS", "48544D4C207B0D0A0942"),
    
    /**  JS */  
    JS("JS", "696B2E71623D696B2E71"), 
     
    /**  Visio */  
    VSD("VSD", "d0cf11e0a1b11ae10000"),
    
    /** WPS文字wps、表格et、演示dps都是一样的 */  
    WPS("WPS", "d0cf11e0a1b11ae10000"),
    
    /** torrent */  
    TORRENT("TORRENT", "6431303A637265617465"), 
    
    /** JSP Archive */  
    JSP("JSP", "3C2540207061676520"),  
    
    /** JAVA Archive */  
    JAVA("JAVA", "7061636B61676520"), 
    
    /** CLASS Archive */  
    CLASS("CLASS", "CAFEBABE0000002E00"), 
    
    /** JAR Archive */  
    JAR("JAR", "504B03040A000000"), 
    
    /** MF Archive */  
    MF("MF", "4D616E69666573742D56"),
 
    /** EXE Archive */  
    EXE("EXE", "4D5A9000030000000400"), 
 
    /** ELF Executable */  
    ELF("ELF", "7F454C4601010100"), 
 
    /** Lotus 123 v1 */  
    WK1("WK1", "2000604060"),
    
    /** Lotus 123 v3 */  
    WK3("WK3", "00001A0000100400"),
    
    /** Lotus 123 v5 */  
    WK4("WK4", "00001A0002100400"), 
 
    /** Lotus WordPro v9 */  
    LWP("LWP", "576F726450726F"), 
    
    /** Sage(sly.or.srt.or.slt;sly;srt;slt) */  
    SLY("SLY", "53520100"), 
```






