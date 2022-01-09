# 断点续传



### 什么是断点续传

断点续传，指的是在上传/下载的过程中，由于网络或其他原因导致上传/下载终断。可以从已经上传或下载的部分开始继续上传下载未完成的部分，而没有必要从头开始上传/下载。用户可以节省时间，提高速度。



![](https://resource.dengzhanyong.com/images/1623984437041.gif)

### 整体思路流程

![](https://resource.dengzhanyong.com/images/1623919809915.jpg)



#### 1. 选择文件-解析文件

先来一个选择文件的input和一个上传按钮

```react
<div>
	<input type="file" ref={InputRef}/>
    <Button type="primary" onClick={uploadFile}>
        上传
    </Button>
</div>
```

选择文件后点击上传

```javascript
async function uploadFile() {
    const file = InputRef.current.files[0];   // 获取选择的文件
    const buffer = await fileParse(file);
}

// 把文件解析为buffer数据
function fileParse(file: any) {
    return new Promise((resolve) => {
        const fileRead = new FileReader();
        fileRead.readAsArrayBuffer(file);
        fileRead.onload = (ev: any) => {
            resolve(ev.target.result);
        };
    });
}

```



#### 2. 生成MD5值

根据文件内容，生成文件的MD5值，用来唯一标识文件，也是后面实现断点续传的基础。

使用 `spark-md5` 包生成MD5 

```javascript
import SparkMD5 from 'spark-md5';

function makeMd5(buffer) {
    const spark = new SparkMD5.ArrayBuffer();
    spark.append(buffer);
    const hash = spark.end();
}
```



#### 3. 文件分片

根据设定的每片数据的大小，计算出当前文件可以分成多少片。

将分片后的数据保存在一个数据中。

```javascript
async function createChunkFile(file: any) {
    if (!file) return;
    const suffix = /\.([0-9a-zA-Z]+)$/i.exec(file.name)[1];  // 获取文件名后缀
    const list = [];
    const count = Math.ceil(file.size / SIZE);  // SIZE指的是规定每一片数据的大小
    const partSize = file.size / count;  // 时间上每片数据的大小

    let cur = 0;
    for (let i = 0; i < count; i++) {
        let item = {
            chunk: file.slice(cur, cur + partSize),
            filename: `${hash}_${i}.${suffix}`,   // 每片文件的名称以MD5_索引值的方式
        };
        cur += partSize;
        list.push(item);
    }
}
```



#### 4. 确定上传索引值

到这里我们已经得到了一个文件分片后的数据列表，但现在还不能开始上传。

有可能此文件在之前已经上传了一些内容，那么只需要上传剩下的内容即可。

因此需要用 MD5 值查询一下应该从第几片开始上传。

```javascript
async function getLoadingFiles() {
    axios.get(`/file/uploaded/count?hash=${hash}`)
        .then((res: any) => {
        if (res.code === 1) {
            const count = res.data.count;
            uploadFn(count);  // 上传文件
        }
    })
}
```



#### 5. 上传文件

现在可以正式开始上传文件了

```javascript
async function uploadFn(startIndex: number = 0) {
    if (list.length === 0) return;
    const requestList: any[] = [];
    list.forEach((item: any, index: number) => {
        const fn = () => {
            let formData = new FormData();
            formData.append('chunk', item.chunk);
            formData.append('filename', item.filename);
            return axios
                .post('/file/upload', formData, {
                headers: { 'Content-Type': 'multipart/form-data' },
            })
                .then((res: any) => {
                const data = res.data;
                if (res.code === 1) {
                    setUploadedIndex(index);
                    setUploadProgress((data.index + 1) * 100 / partList.current.length);
                }
            })
                .catch(function () {
                setAbort(true);
                message.error('上传失败')
            });
        };
        requestList.push(fn);
    });
    uploadSend(startIndex, requestList);
}

// 上传单个切片
async function uploadSend(index: number, requestList: any) {
    if (abortRef.current) return;
    if (index >= requestList.length) {
        uploadComplete();  // 上传完成
        return;
    }
    await requestList[index]();
    uploadSend( ++index, requestList);
}
```



#### 6.  上传完成

当列表内的所有数据都上传完成后还没有结局，需要通知后端可以合并文件了。

```javascript
// 上传完成
async function uploadComplete() {
    let result: any = await axios.get(`/file/upload/finish?hash=${hash.current}`);
    if (result.code === 1) {
        message.success('上传成功');
    }
}
```



**到这了前端的内容就全部完成了，贴一下完整代码** 

```react
import { useState, useRef, useEffect } from 'react';
import { Button, message, Progress } from 'antd';
import SparkMD5 from 'spark-md5';
import { formatFileSizeUnit } from '@/common';
import axios from '@/axios';
import { PlayCircleFilled, PauseCircleFilled } from '@ant-design/icons';
import './uploadingPanel.less';

const SIZE = 1024 * 1024 * 2;

const UploadingPanel = (props: any) => {

    const {} = props;

    const [abort, setAbort] = useState<boolean>(false);
    const [uploadProgress, setUploadProgress] = useState<number>(0);  // 上传进度
    const [uploadedIndex, setUploadedIndex] = useState(0);  // 上传进度
    const [infoMsg, setInfoMsg] = useState<string>('');   // 
    // const [fileInfo, setFileInfo] = useState<any>({});
    const InputRef = useRef<any>(null);
    const hash = useRef<any>(null);  // 文件hash值
    const partList = useRef<any>([]);   // 分片后的文件
    const abortRef = useRef<any>(false);

    // 上传文件
    async function uploadFile() {
        setUploadProgress(0);
        setInfoMsg('文件分解中');
        const file = InputRef.current.files[0];
        await createChunkFile(file);
    }

    //转换文件类型（解析为BUFFER数据）
    function fileParse(file: any) {
        return new Promise((resolve) => {
            const fileRead = new FileReader();
            fileRead.readAsArrayBuffer(file);
            fileRead.onload = (ev: any) => {
                resolve(ev.target.result);
            };
        });
    }

    async function createChunkFile(file: any) {
        if (!file) return;
        const buffer = await fileParse(file);
        const spark = new SparkMD5.ArrayBuffer();
        spark.append(buffer);
        hash.current = spark.end();
        const suffix = /\.([0-9a-zA-Z]+)$/i.exec(file.name)[1];
        const list = [];
        const count = Math.ceil(file.size / SIZE);
        const partSize = file.size / count;

        let cur = 0;
        for (let i = 0; i < count; i++) {
            let item = {
                chunk: file.slice(cur, cur + partSize),
                filename: `${hash.current}_${i}.${suffix}`,
            };
            cur += partSize;
            list.push(item);
        }
        partList.current = list;
        getLoadingFiles();
    }

    async function getLoadingFiles() {
        axios.get(`/file/uploaded/count?hash=${hash.current}`)
            .then((res: any) => {
                if (res.code === 1) {
                    const count = res.data.count;
                    setInfoMsg('文件上传中');
                    setUploadProgress((count * 100 / partList.current.length).toFixed(2));
                    uploadFn(count);
                }
            })
    }

    async function uploadFn(startIndex: number = 0) {
        if (partList.current.length === 0) return;
        abortRef.current = false;
        const requestList: any[] = [];
        partList.current.forEach((item: any, index: number) => {
            const fn = () => {
                let formData = new FormData();
                formData.append('chunk', item.chunk);
                formData.append('filename', item.filename);
                return axios
                    .post('/file/upload', formData, {
                        headers: { 'Content-Type': 'multipart/form-data' },
                    })
                    .then((res: any) => {
                        const data = res.data;
                        if (res.code === 1) {
                            setUploadedIndex(index);
                            setUploadProgress((data.index + 1) * 100 / partList.current.length);
                        }
                    })
                    .catch(function () {
                        setAbort(true);
                        message.error('上传失败')
                    });
            };
            requestList.push(fn);
        });
        uploadSend(startIndex, requestList);
    }

    // 上传单个切片
    async function uploadSend(index: number, requestList: any) {
        if (abortRef.current) return;
        if (index >= requestList.length) {
            uploadComplete();
            return;
        }
        requestList[index] ? await requestList[index]() : setInfoMsg('');
        uploadSend( ++index, requestList);
    }

    // 上传完成
    async function uploadComplete() {
        let result: any = await axios.get(`/file/upload/finish?hash=${hash.current}`);
        if (result.code === 1) {
            message.success('上传成功');
            setInfoMsg('上传完成');
        }
    }

    // 选择文件
    function changeFile() {
        const file = InputRef.current.files[0];
        setFileInfo(file || {});
    }

    return (
        <div className="uploading-panel">
            <input type="file" onChange={changeFile} ref={InputRef}/>
            {infoMsg && (
                <span>【{infoMsg}】</span>
            )}
            {fileInfo.size && (
                <span>{formatFileSizeUnit(fileInfo.size)}</span>
            )}
            <Button type="primary" onClick={uploadFile}>
                上传
            </Button>
            <Button
                type="primary"
                shape="circle"
                icon={abort ? <PlayCircleFilled /> : <PauseCircleFilled />}
                onClick={() => {
                    abortRef.current = !abort;
                    abort && uploadFn(uploadedIndex + 1)
                    setAbort(!abort)
                }}
            />
            <div>
                <Progress percent={uploadProgress} status="active"/>
            </div>
        </div>
    );
};

export default UploadingPanel;

```



### 接口实现

现在来看看后端接口的实现，后端我使用的 `koa`

#### 1. 查询某个文件已经上传多少片了

前端在分片时，对数据是按照索引顺序进行上传的，并且只有上一片上传成功后才会开始下一片的上传。

那么我们只需要计算当前已经上传分片的数量，返回给前端，前端只需要从相应的索引出上传即可。

```javascript
router.get('/uploaded/count', async (ctx, next) => {
  const {
    hash
  } = ctx.query;
  const filePath = `${uploadDir}${hash}`;
  const fileList = (fs.existsSync(filePath) && fs.readdirSync(filePath)) || [];
  ctx.body = {
    code: 1,
    data: {
      count: fileList.length
    }
  }
})
```



#### 2. 接收上传文件

```javascript
router.post('/upload', async (ctx, next) => {
  const file = ctx.request.files.chunk // 获取上传文件
  const {
    filename,
  } = ctx.request.body;
  const reader = fs.createReadStream(file.path);
  const [hash, suffix] = filename.split('_');
  const folder = uploadDir + hash;
  !fs.existsSync(folder) && fs.mkdirSync(folder);
  const filePath =  `${folder}/${filename}`;
  const upStream = fs.createWriteStream(filePath);
  reader.pipe(upStream);
  ctx.body = await new Promise((resolve, reject) => {
    reader.on('error', () => {
      reject({
        code: 0,
        massage: '上传失败',
      })
    })
    reader.on('close', () => {    
      resolve({
        code: 1,
        massage: '上传成功',
        data: {
          hash,
          index: Number(suffix.split('.')[0])
        }
      })
    })
  })
  
})
```



#### 3. 合并文件

```javascript
router.get('/upload/finish', async (ctx, next) => {
  const {
    hash
  } = ctx.query;
  const filePath = `${uploadDir}${hash}`;
  const fileList = fs.readdirSync(filePath);
  let suffix = '';
  fileList.sort((a, b) => {
      let reg = /_(\d+)/;
      return reg.exec(a)[1] - reg.exec(b)[1];
    }).forEach((item) => {
      suffix = /\.([0-9a-zA-Z]+)$/.exec(item)[1] || null;
      fs.appendFileSync(`${path.join(__dirname, '../public/file/')}${hash}.${suffix}`, fs.readFileSync(`${filePath}/${item}`));
      fs.existsSync(`${filePath}/${item}`) && fs.unlinkSync(`${filePath}/${item}`);
    });
    fs.existsSync(filePath) && fs.unlinkSync(filePath);
  ctx.body = {
    code: 1,
    msg: '上传成功'
  }
})
```



到现在为止就已经完全实现了断点续传的功能，在上传过程中可以手动暂停/继续。

即使是刷新页面后，当我们重新选择未上传完成的文件时，也可以从上次断开的地方继续上传。









