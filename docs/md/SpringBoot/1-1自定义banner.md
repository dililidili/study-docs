# 1.1、自定义 banner
可以通过在 classpath 下添加一个 banner.txt 文件
<br />
![banner文件位置](https://raw.githubusercontent.com/dililidili/study-docs/main/docs/img/SpringBoot/custom_banner.jpg)
<br />
推荐一个生成自定义图形的网址:
[生成自定义banner图标](https://bootschool.net/ascii "生成自定义banner图标")

提供一个现成的banner
    
        ${AnsiColor.BRIGHT_YELLOW}
        ////////////////////////////////////////////////////////////////////
        //                          _ooOoo_                               //
        //                         o8888888o                              //
        //                         88" . "88                              //
        //                         (| ^_^ |)                              //
        //                         O  =  /O                              //
        //                      ____/`---'\____                           //
        //                    .'  \|     |//  `.                         //
        //                   /  \|||  :  |||//                          //
        //                  /  _||||| -:- |||||-                         //
        //                  |   | \  -  /// |   |                       //
        //                  | \_|  ''---/''  |   |                       //
        //                    .-\__  `-`  ___/-. /                       //
        //                ___`. .'  /--.--  `. . ___                     //
        //              ."" '<  `.___\_<|>_/___.'  >'"".                  //
        //            | | :  `- \`.;` _ /`;.`/ - ` : | |                 //
        //               `-.   \_ __ /__ _/   .-` /  /                 //
        //      ========`-.____`-.___\_____/___.-`____.-'========         //
        //                           `=---='                              //
        //      ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^        //
        //            佛祖保佑       永不宕机      永无BUG                //
        ////////////////////////////////////////////////////////////////////

## banner.txt配置
- ${AnsiColor.BRIGHT_YELLOW}：设置控制台中输出内容的颜色
- ${application.version}：用来获取MANIFEST.MF文件中的版本号
- ${application.formatted-version}：格式化后的${application.version}版本信息
- ${spring-boot.version}：SpringBoot的版本号
- ${spring-boot.formatted-version}：格式化后的${spring-boot.version}版本信息



