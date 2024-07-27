由
https://github.com/F6JO/mPaas-frida-hook/blob/main/frida.js
改动

```
function utf8ByteToUnicodeStr(utf8Bytes) {
    var unicodeStr = "";
    for (var pos = 0; pos < utf8Bytes.length;) {
        var flag = utf8Bytes[pos];
        var unicode = 0;
        if ((flag >>> 7) === 0) {
            unicodeStr += String.fromCharCode(utf8Bytes[pos]);
            pos += 1;
        } else if ((flag & 0xFC) === 0xFC) {
            unicode = (utf8Bytes[pos] & 0x3) << 30;
            unicode |= (utf8Bytes[pos + 1] & 0x3F) << 24;
            unicode |= (utf8Bytes[pos + 2] & 0x3F) << 18;
            unicode |= (utf8Bytes[pos + 3] & 0x3F) << 12;
            unicode |= (utf8Bytes[pos + 4] & 0x3F) << 6;
            unicode |= (utf8Bytes[pos + 5] & 0x3F);
            unicodeStr += String.fromCharCode(unicode);
            pos += 6;
        } else if ((flag & 0xF8) === 0xF8) {
            unicode = (utf8Bytes[pos] & 0x7) << 24;
            unicode |= (utf8Bytes[pos + 1] & 0x3F) << 18;
            unicode |= (utf8Bytes[pos + 2] & 0x3F) << 12;
            unicode |= (utf8Bytes[pos + 3] & 0x3F) << 6;
            unicode |= (utf8Bytes[pos + 4] & 0x3F);
            unicodeStr += String.fromCharCode(unicode);
            pos += 5;
        } else if ((flag & 0xF0) === 0xF0) {
            unicode = (utf8Bytes[pos] & 0xF) << 18;
            unicode |= (utf8Bytes[pos + 1] & 0x3F) << 12;
            unicode |= (utf8Bytes[pos + 2] & 0x3F) << 6;
            unicode |= (utf8Bytes[pos + 3] & 0x3F);
            unicodeStr += String.fromCharCode(unicode);
            pos += 4;
        } else if ((flag & 0xE0) === 0xE0) {
            unicode = (utf8Bytes[pos] & 0x1F) << 12;
            unicode |= (utf8Bytes[pos + 1] & 0x3F) << 6;
            unicode |= (utf8Bytes[pos + 2] & 0x3F);
            unicodeStr += String.fromCharCode(unicode);
            pos += 3;
        } else if ((flag & 0xC0) === 0xC0) { //110
            unicode = (utf8Bytes[pos] & 0x3F) << 6;
            unicode |= (utf8Bytes[pos + 1] & 0x3F);
            unicodeStr += String.fromCharCode(unicode);
            pos += 2;
        } else {
            unicodeStr += String.fromCharCode(utf8Bytes[pos]);
            pos += 1;
        }
    }
    return unicodeStr;
}


function stringToByte(str) {
    var javaString = Java.use('java.lang.String');
    var bytes = [];
    bytes = javaString.$new(str).getBytes();
    return bytes;
}


function uuid() {
    var s = [];
    var hexDigits = "0123456789abcdef";
    for (var i = 0; i < 36; i++) {
        s[i] = hexDigits.substr(Math.floor(Math.random() * 0x10), 1);
    }
    s[14] = "4"; // bits 12-15 of the time_hi_and_version field to 0010
    s[19] = hexDigits.substr((s[19] & 0x3) | 0x8, 1); // bits 6-7 of the clock_seq_hi_and_reserved to 01
    s[8] = s[13] = s[18] = s[23] = "-";

    var uuid = s.join("");
    uuid = uuid.split("-").join("");
    return uuid;
}

Java.perform(function () {

        var JsonSerializerV2 = Java.use('com.alipay.mobile.common.rpc.protocol.json.JsonSerializerV2');

        JsonSerializerV2.packet.implementation = function () {
            console.log();
            console.log("------------------JsonSerializerV2---------------------------------");
            var uuidx = uuid();
            var OldReqData = utf8ByteToUnicodeStr(this.packet());
           
            // send({type: 'REQ', data: OldReqData});
            // var typexx;
            // var string_to_recv;
            // typexx = received_json_object['type'];
            // string_to_recv = received_json_object['payload'];

            // if (typexx === 'NEW_REQ') {
            //     console.log(uuidx + ": NewREQ: " + string_to_recv)
            //     console.log("------------------JsonSerializerV2 end---------------------------------");
            OldReqData = OldReqData.replace('"level":"3"','"level":"3\'"')
            console.log(OldReqData)
            // OldReqData = OldReqData.replace(/"imgStr":"(.*?)"/,'"imgStr":"PHNjcmlwdD5hbGVydCgxKTwvc2NyaXB0Pg=="')
            
                return stringToByte(OldReqData);
            // }

            // console.log("!!!!!!!!!!!!!!!!!!JsonSerializerV2 NOT NEW_REQ!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!1")
            // return this.packet();
        }

        var SignJsonSerializer = Java.use('com.alipay.mobile.common.rpc.protocol.json.SignJsonSerializer');
        SignJsonSerializer.packet.implementation = function () {
            console.log();
            console.log("------------------SignJsonSerializer---------------------------------");
            var uuidx = uuid();
            var OldReqData = utf8ByteToUnicodeStr(this.packet());
            console.log(OldReqData)
            // send({type: 'REQ', data: OldReqData});
            // var typexx;
            // var string_to_recv;
            // typexx = received_json_object['type'];
            // string_to_recv = received_json_object['payload'];

            // if (typexx === 'NEW_REQ') {
            //     console.log(uuidx + ": NewREQ: " + string_to_recv)
            //     console.log("------------------SignJsonSerializer end---------------------------------");
            //     return stringToByte(string_to_recv + "!!!uuidx:" +uuidx + "!!!");
            // }

            // console.log("!!!!!!!!!!!!!!!!!!SignJsonSerializer NOT NEW_REQ!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!1")
            return this.packet();
        }

        var HttpCaller = Java.use('com.alipay.mobile.common.rpc.transport.http.HttpCaller');
        HttpCaller.b.overload('com.alipay.mobile.common.transport.http.HttpUrlRequest').implementation = function (a){
            var uuidchuanzu = utf8ByteToUnicodeStr(a.getReqData()).match(/!!!uuidx:(.*?)!!!/g);
            var uuidchuan = "";
            if (uuidchuanzu != null){
                uuidchuan = uuidchuanzu[0];
            }
            var uuidx = uuidchuan.replace("!!!uuidx:","").replace("!!!","");

            a.setReqData(stringToByte(utf8ByteToUnicodeStr(a.getReqData()).replace(uuidchuan,"")));
            var resp = this.b(a);
            console.log(resp.getClass());
            var respStr = utf8ByteToUnicodeStr(resp.getResData());
            console.log("\n======response======\n"+respStr)

            // send({type: 'RESP', data: respStr});
            // var typexx;
            // var string_to_recv;
            // typexx = received_json_object['type'];
            // string_to_recv = received_json_object['payload'];
            
            // if (typexx === 'NEW_RESP') {
            //     console.log(uuidx+": NewREP: " + string_to_recv)
            //     resp.setResData(stringToByte(string_to_recv))
            //     return resp
            // }
            return resp


        }

        var RpcInvoker = Java.use('com.alipay.mobile.common.rpc.RpcInvoker');
        RpcInvoker.a.overload('java.lang.reflect.Method','java.lang.String','[B','com.alipay.mobile.common.rpc.transport.InnerRpcInvokeContext','com.alipay.mobile.common.rpc.transport.http.HttpCaller').implementation = function (a,b,c,d,e){
            // console.log("RpcInvokerRpcInvokerRpcInvoker")
            var uuidchuanzu = utf8ByteToUnicodeStr(c).match(/!!!uuidx:(.*?)!!!/g);
            if (uuidchuanzu != null){
                var uuidchuan = uuidchuanzu[0];
                var newc = stringToByte(utf8ByteToUnicodeStr(c).replace(uuidchuan,""))
                // console.log(utf8ByteToUnicodeStr(newc))
                this.a(a,b,newc,d,e);
            }else {
                this.a(a,b,c,d,e)
            }

            // this.a(a,b,c,d,e);
        }




        let Response = Java.use("com.alipay.mobile.common.transport.Response");
Response["getResData"].implementation = function () {
    // console.log(`Response.getResData is called`);
    let result = this["getResData"]();
    // console.log(`${}`);
    // result = stringToByte(utf8ByteToUnicodeStr(result).replace("WvRoplfw4aaTwQkxdmRQsA==","JIkIkVByJPPdrQKGG%2BihFg=="))
    return result;
};
    }
);
```
