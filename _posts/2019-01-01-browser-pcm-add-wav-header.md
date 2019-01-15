---
layout: post
title: 【音视频编程】在浏览器中播放pcm音频的案例
date:   2019-01-01 20:30:00 +0800
categories: pcm
tag: [browser]
---


* content
{:toc}


## 前言

最近在整理音视频编程的知识，回忆起半年多，有一次需求是在后台播放某来源的 pcm 文件，当时处理方法用了点技巧，记录下来

背景：业务需求，在web后台里播放 pcm 文件，文件不大(约300KB)

## 如何播放

浏览器是无法直接播放 pcm 音频的，因为 pcm 是比较原始的音频格式:

PCM（Puls Code Modulation）全称脉码调制录音，PCM录音就是将声音的模拟信号表示成0,1标识的数字信号，未经任何编码和压缩处理，所以可以认为PCM是未经压缩的音频原始格式。PCM格式文件中不包含头部信息，播放器无法知道采样率，声道数，采样位数，音频数据大小等信息，导致无法播放。

### 如何让浏览器识别 pcm

浏览器可以播放另一种音频格式：WAV格式全称为WAVE，前面提到只需要在PCM文件的前面添加WAV文件头，就可以生成WAV格式文件

所以我的解决方法是给 pcm 添加 wav header，接下来就是 browser javascript 的实践编码了

### javascript 如何处理文件流

js 在处理文件流、网络数据，常用到 ArrayBuffer 类型，关于这个类型的API调用方法，需要事先多了解。


- 第一步，ajax异步获取网络 pcm 文件的 ArrayBuffer

```javascript
const getWebFileArrayBuffer = async (url) => {
  return await fetch(url).then(response => response.arrayBuffer())
}
```

- 第二步，对获取的 pcm 文件流 ArrayBuffer 添加 wav header，下面以代码注释为大家解释

```javascript
const getWebPcm2WavArrayBuffer = async (url) => {
  const bytes = await getWebFileArrayBuffer(url)
  return addWavHeader(bytes, 16000, 16, 1)
}

const addWavHeader = function (samples, sampleRateTmp, sampleBits, channelCount) {
  let dataLength = samples.byteLength
  let buffer = new ArrayBuffer(44 + dataLength)
  let view = new DataView(buffer)
  function writeString (view, offset, string) {
    for (let i = 0; i < string.length; i++) {
      view.setUint8(offset + i, string.charCodeAt(i))
    }
  }

  let offset = 0
  /* 资源交换文件标识符 */
  writeString(view, offset, 'RIFF'); offset += 4
  /* 下个地址开始到文件尾总字节数,即文件大小-8 */
  view.setUint32(offset, /* 32 */ 36 + dataLength, true); offset += 4
  /* WAV文件标志 */
  writeString(view, offset, 'WAVE'); offset += 4
  /* 波形格式标志 */
  writeString(view, offset, 'fmt '); offset += 4
  /* 过滤字节,一般为 0x10 = 16 */
  view.setUint32(offset, 16, true); offset += 4
  /* 格式类别 (PCM形式采样数据) */
  view.setUint16(offset, 1, true); offset += 2
  /* 通道数 */
  view.setUint16(offset, channelCount, true); offset += 2
  /* 采样率,每秒样本数,表示每个通道的播放速度 */
  view.setUint32(offset, sampleRateTmp, true); offset += 4
  /* 波形数据传输率 (每秒平均字节数) 通道数×每秒数据位数×每样本数据位/8 */
  view.setUint32(offset, sampleRateTmp * channelCount * (sampleBits / 8), true); offset += 4
  /* 快数据调整数 采样一次占用字节数 通道数×每样本的数据位数/8 */
  view.setUint16(offset, channelCount * (sampleBits / 8), true); offset += 2
  /* 每样本数据位数 */
  view.setUint16(offset, sampleBits, true); offset += 2
  /* 数据标识符 */
  writeString(view, offset, 'data'); offset += 4
  /* 采样数据总数,即数据总大小-44 */
  view.setUint32(offset, dataLength, true); offset += 4
  function floatTo32BitPCM (output, offset, input) {
    input = new Int32Array(input)
    for (let i = 0; i < input.length; i++, offset += 4) {
      output.setInt32(offset, input[i], true)
    }
  }
  function floatTo16BitPCM (output, offset, input) {
    input = new Int16Array(input)
    for (let i = 0; i < input.length; i++, offset += 2) {
      output.setInt16(offset, input[i], true)
    }
  }
  function floatTo8BitPCM (output, offset, input) {
    input = new Int8Array(input)
    for (let i = 0; i < input.length; i++, offset++) {
      output.setInt8(offset, input[i], true)
    }
  }
  if (sampleBits == 16) {
    floatTo16BitPCM(view, 44, samples)
  } else if (sampleBits == 8) {
    floatTo8BitPCM(view, 44, samples)
  } else {
    floatTo32BitPCM(view, 44, samples)
  }
  return view.buffer
}
```

- 第三步，在浏览器播放 pcm 转出的 wav 的文件流 ArrayBuffer

+ 先转成 base64 格式

```javascript
const getWebPcm2WavBase64 = async (url) => {
  let bytes = await getWebPcm2WavArrayBuffer(url)
  return `data:audio/wav;base64,${btoa(new Uint8Array(bytes).reduce((data, byte) => {
    return data + String.fromCharCode(byte)
  }, ''))}`
}

```

+ 将 base64 字符串放入`<audio>`组件中，这里以 `react/ant design` 的组件为例，封装一个方法

```javascript
const playWebPcm = async (url) => {
    try {
    let pcmBase64 = await fileServer.getWebPcm2WavBase64(url)
    Modal.info({
        title: '播放音频',
        content: (
        <audio controls src={pcmBase64} type="audio/wav" autoPlay />
        ),
        onOk () {},
        okText: '关闭',
    })
    } catch (err) {
    console.error(err)
    message.error('预载音频文件失败')
    }
}
```

## Reference

[音频格式简介和PCM转换成WAV，Java版本](https://blog.csdn.net/u010126792/article/details/86493494)