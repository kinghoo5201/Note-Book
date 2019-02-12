## 一个分页函数示例

前后端通用，如需置于前端需要转es5
       
```html
<!DOCTYPE html>
<html lang="en">

<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <meta http-equiv="X-UA-Compatible" content="ie=edge">
  <title>Document</title>
</head>

<body>

  <script>
    const pagination = (opt) => {
      const defaultOpt = {
        outerClass: 'pagination',// 外层类名
        itemClass: 'pagination-item',//item类名
        activeClass: 'on',//选中类名
        total: 1,//总页数
        page: 1,//当前页数
        length: 5,//展示长度
        target: '_self',//跳转类型
        baseUrl: '#{page}',//链接
        useDefaultCss: true,//如果需要自定义style需关闭
        preText: '上一页',//上页文案
        nexText: '下一页',//下页文案
        indecator: '•••',//省略号文案
        showIpt: false,
        inputInfo: '跳至{input}页',
      };
      const option = {
        ...defaultOpt,
        ...opt
      };
      if (option.total <= 1) {
        return '';
      }
      const wrapper = `
      {style}
      <div class="${option.outerClass}">
      {content}
      </div>
      `;
      let content = '';
      let style = '';
      const btnItem = `<a class="{class}" href="{url}" target="${option.target}">{text}</a>`;
      if (option.useDefaultCss) {
        style = `
        <style>
        .${option.outerClass}{
          display:block;
          padding:10px 0;
          text-align:center;
          color:rgba(0,0,0,0.25);
          font-size:14px;
        }
        .${option.outerClass} .${option.itemClass}{
          display:inline-block;
          border:1px solid #d9d9d9;
          color:rgba(0,0,0,0.65);
          border-radius:4px;
          margin:0 4px;
          text-decoration:none;
          cursor:pointer;
          height:32px;
          line-height:32px;
          user-select:none;
          padding:0 10px;
          font-size:14px;
        }
        .${option.outerClass} .${option.itemClass}:hover,.${option.outerClass} .${option.itemClass}.${option.activeClass}{
          -webkit-transition: all .3s;
          transition: all .3s;
          border-color: #1890ff;
          color:#1890ff;
        }
        .${option.outerClass} .${option.itemClass}.${option.activeClass}{
          pointer-events:none;
        }
        .${option.outerClass}-input{
          color:rgba(0,0,0,0.65);
          display:inline-block;
          margin-left:10px;
        }
        .${option.outerClass}-input input{
          position: relative;
          display: inline-block;
          padding: 4px 11px;
          height: 32px;
          font-size: 14px;
          line-height: 1.5;
          color: rgba(0,0,0,0.65);
          background-color: #fff;
          background-image: none;
          border: 1px solid #d9d9d9;
          border-radius: 4px;
          -webkit-transition: all .3s;
          transition: all .3s;
          margin: 0 8px;
          width: 50px;
          box-sizing: border-box;
        }
        .${option.outerClass}-input input:hover{
          border-color: #40a9ff;
          border-right-width: 1px !important;
        }
        .${option.outerClass}-input input:focus{
          border-color: #40a9ff;
          outline: 0;
          -webkit-box-shadow: 0 0 0 2px rgba(24,144,255,0.2);
          box-shadow: 0 0 0 2px rgba(24,144,255,0.2);
          border-right-width: 1px !important;
        }
        </style>
        `;
      }
      const loopArr = new Array(option.total).join(',').split(',');
      const isFirst = (1 === option.page);
      const isLast = (option.total === option.page);
      const hasPLBtn = option.total > option.length;
      content += hasPLBtn && !isFirst ?
        btnItem.replace(/\{class\}/g, option.itemClass)
          .replace(/\{text\}/g, option.preText)
          .replace(/\{url\}/g, option.baseUrl.replace(/\{page\}/g, option.page - 1))
        : '';
      for (const i in loopArr) {
        const index = Number(i) + 1;
        const isCurrent = index === option.page;
        let isInd = false;
        const midLen = option.length - 2;
        const mid = option.page;
        let preMidLen = Math.floor(midLen / 2);
        let nexMidLen = midLen - preMidLen - 1;
        const scale = [];
        if (option.total > option.length) {
          if (mid - preMidLen < 2) {
            preMidLen = mid - 2;
            nexMidLen = midLen - preMidLen - 1;
          }
          if (mid + nexMidLen >= option.total) {
            nexMidLen = option.total - mid - 1;
            preMidLen = midLen - nexMidLen - 1;
          }
          scale.push(mid - preMidLen);
          scale.push(mid + nexMidLen);
          if (index !== 1 || index !== option.total) {
            isInd = true;
          }
        }
        if (isInd) {
          if ((index === 2 || index === option.total - 1) && (index < scale[0] || index > scale[1])) {
            content += option.indecator;
            continue;
          }
          if (index !== 1 && index !== option.total && (index < scale[0] || index > scale[1])) {
            continue;
          }
        }
        content += btnItem.replace(/\{class\}/g, option.itemClass + (isCurrent ? ` ${option.activeClass}` : ''))
          .replace(/\{text\}/g, index)
          .replace(/\{url\}/g, option.baseUrl.replace(/\{page\}/g, index))
      }
      content += hasPLBtn && !isLast ?
        btnItem.replace(/\{class\}/g, option.itemClass)
          .replace(/\{text\}/g, option.nexText)
          .replace(/\{url\}/g, option.baseUrl.replace(/\{page\}/g, option.page + 1))
        : '';
      const input = `
      <div class="${option.outerClass}-input">
        ${option.inputInfo.replace(/\{input\}/, `
        <input type="text" onKeyDown="javascript:if(event.keyCode===13){var page=Number(this.value);if(!page||page===${option.page}||page<=0||page>${option.total}){return;}window.location.href='${option.baseUrl}'.replace(/\{page\}/g,page);}else{var value=this.value;value=value.replace(/\\D/gi,'').replace(/\\s/g,'');this.value=value;}" onKeyUp="javascript:var value=this.value;value=value.replace(/\\D/gi,'').replace(/\\s/g,'');this.value=value;"/>
        `)}
      </div>
      `;
      option.showIpt && (content += input);
      const result = wrapper.replace(/\{content\}/g, content).replace(/\{style\}/g, style);
      return result;
    };
    document.write(pagination({ total: 13, page: 2, showIpt: true }));
  </script>
</body>

</html>
```
