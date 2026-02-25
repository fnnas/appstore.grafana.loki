# Grafana Loki for 飞牛NAS (fnOS)

本项目用于将 [Grafana Loki](https://github.com/grafana/loki) 打包为飞牛NAS应用商店的安装包。

## 关于 Loki

Loki 是一个受 Prometheus 启发的、具有横向扩展性、高可用性及多租户特性的日志聚合系统。  
它旨在实现极高的成本效益和易用性。与其他日志系统不同，Loki 不对日志内容进行全文索引，而是为每个日志流的标签集创建索引，这使其非常高效。

## 支持架构
飞牛官方对架构名称的定义与约定俗成的名称不一致，以下均列出

| 实际架构            | 飞牛叫法 |
|-----------------|------|
| x86_64 (amd64)  | x86  |
| aarch64 (arm64) | arm  |

## 目录结构
以下是主要的目录，没有列出全部来  
```
.
├── .github/workflows/     # GitHub Actions 自动化打包流程
│   ├── main.yml           # 主工作流
│   ├── pack_amd64.yml     # amd64 架构打包工作流
│   └── pack_arm64.yml     # arm64 架构打包工作流
├── app/                   # 应用文件
│   ├── config.loki.template.yaml  # Loki 配置模板
│   └── ui/images/         # 应用图标
├── package/               # 预制菜包
│   ├── manifest           # 应用清单
│   ├── cmd/               # 启动/停止脚本
│   │   └── main           # 主控制脚本
│   └── config/            # 权限与资源配置
└── ui/                    # UI 文件目录
```

## 安装后配置

### 数据存储路径

所有持久化数据存储在：
```
/var/apps/grafana.loki/shares/grafana.loki/
```

包含：
- `config.loki.yaml` - Loki 运行配置文件
- `chunks/` - 日志数据块存储
- `index_*/` - 索引文件

### 默认端口

- HTTP API 端口：`3100`

### 配置文件

首次启动时会自动从模板复制配置文件。如需修改配置，编辑：
```
/var/apps/grafana.loki/shares/grafana.loki/config.loki.yaml
```
对应webui的`文件管理-应用文件-grafana.loki`目录

修改后重启服务生效。

## 使用方法

### 推送日志

可以使用 Promtail、Grafana Agent 或其他兼容客户端向 Loki 推送日志：  
以下是使用shell进行验证测试  
```bash
curl -X POST "http://<NAS_IP>:3100/loki/api/v1/push" \
  -H "Content-Type: application/json" \
  -d '{
    "streams": [{
      "stream": { "job": "test" },
      "values": [["'$(date +%s)000000000'", "Hello from fnOS!"]]
    }]
  }'
```

### 查询日志
正常情况下不会使用接口查询，但可以使用接口验证日志是否被正确投递接受  
```bash
curl -G "http://<NAS_IP>:3100/loki/api/v1/query_range" \
  --data-urlencode 'query={job="test"}'
```
可见返回出现刚刚投递的`Hello from fnOS!`日志
```json
{"status":"success","data":{"resultType":"streams","result":[{"stream":{"detected_level":"unknown","job":"test","service_name":"test"},"values":[["1771992982000000000","Hello from fnOS!"]]}],"stats":{"summary":{"bytesProcessedPerSecond":2249,"linesProcessedPerSecond":93,"totalBytesProcessed":24,"totalLinesProcessed":1,"execTime":0.010669,"queueTime":0.001127,"subqueries":0,"totalEntriesReturned":1,"splits":2,"shards":2,"totalPostFilterLines":1,"totalStructuredMetadataBytesProcessed":8},"querier":{"store":{"totalChunksRef":0,"totalChunksDownloaded":0,"chunksDownloadTime":0,"queryReferencedStructuredMetadata":false,"queryUsedV2Engine":false,"chunk":{"headChunkBytes":0,"headChunkLines":0,"decompressedBytes":0,"decompressedLines":0,"compressedBytes":0,"totalDuplicates":0,"postFilterLines":0,"headChunkStructuredMetadataBytes":0,"decompressedStructuredMetadataBytes":0},"chunkRefsFetchTime":0,"congestionControlLatency":0,"pipelineWrapperFilteredLines":0,"dataobj":{"prePredicateDecompressedRows":0,"prePredicateDecompressedBytes":0,"prePredicateDecompressedStructuredMetadataBytes":0,"postPredicateRows":0,"postPredicateDecompressedBytes":0,"postPredicateStructuredMetadataBytes":0,"postFilterRows":0,"pagesScanned":0,"pagesDownloaded":0,"pagesDownloadedBytes":0,"pageBatches":0,"totalRowsAvailable":0,"totalPageDownloadTime":0}}},"ingester":{"totalReached":2,"totalChunksMatched":1,"totalBatches":3,"totalLinesSent":1,"store":{"totalChunksRef":0,"totalChunksDownloaded":0,"chunksDownloadTime":0,"queryReferencedStructuredMetadata":false,"queryUsedV2Engine":false,"chunk":{"headChunkBytes":24,"headChunkLines":1,"decompressedBytes":0,"decompressedLines":0,"compressedBytes":0,"totalDuplicates":0,"postFilterLines":1,"headChunkStructuredMetadataBytes":8,"decompressedStructuredMetadataBytes":0},"chunkRefsFetchTime":628781,"congestionControlLatency":0,"pipelineWrapperFilteredLines":0,"dataobj":{"prePredicateDecompressedRows":0,"prePredicateDecompressedBytes":0,"prePredicateDecompressedStructuredMetadataBytes":0,"postPredicateRows":0,"postPredicateDecompressedBytes":0,"postPredicateStructuredMetadataBytes":0,"postFilterRows":0,"pagesScanned":0,"pagesDownloaded":0,"pagesDownloadedBytes":0,"pageBatches":0,"totalRowsAvailable":0,"totalPageDownloadTime":0}}},"cache":{"chunk":{"entriesFound":0,"entriesRequested":0,"entriesStored":0,"bytesReceived":0,"bytesSent":0,"requests":0,"downloadTime":0,"queryLengthServed":0},"index":{"entriesFound":0,"entriesRequested":0,"entriesStored":0,"bytesReceived":0,"bytesSent":0,"requests":0,"downloadTime":0,"queryLengthServed":0},"result":{"entriesFound":0,"entriesRequested":0,"entriesStored":0,"bytesReceived":0,"bytesSent":0,"requests":0,"downloadTime":0,"queryLengthServed":0},"statsResult":{"entriesFound":1,"entriesRequested":1,"entriesStored":0,"bytesReceived":175,"bytesSent":0,"requests":1,"downloadTime":13126,"queryLengthServed":2317000000000},"volumeResult":{"entriesFound":0,"entriesRequested":0,"entriesStored":0,"bytesReceived":0,"bytesSent":0,"requests":0,"downloadTime":0,"queryLengthServed":0},"seriesResult":{"entriesFound":0,"entriesRequested":0,"entriesStored":0,"bytesReceived":0,"bytesSent":0,"requests":0,"downloadTime":0,"queryLengthServed":0},"labelResult":{"entriesFound":0,"entriesRequested":0,"entriesStored":0,"bytesReceived":0,"bytesSent":0,"requests":0,"downloadTime":0,"queryLengthServed":0},"instantMetricResult":{"entriesFound":0,"entriesRequested":0,"entriesStored":0,"bytesReceived":0,"bytesSent":0,"requests":0,"downloadTime":0,"queryLengthServed":0}},"index":{"totalChunks":0,"postFilterChunks":0,"shardsDuration":0,"usedBloomFilters":false}}}}
```
### 配合 Grafana 使用

1. 在 Grafana 中添加 Loki 数据源
2. URL 默认设置为：`http://<NAS_IP>:3100`
3. 即可在 Grafana 中查询和可视化日志

## 安全提示

默认配置下 Loki 未启用认证，请注意至少使用一项安全措施：
- 正确配置防火墙规则以限制访问
- 修改配置启用认证
- 避免将端口直接暴露到公网

不然你的日志内容会被泄露到公网，若日志含有敏感数据(如授权信息、token等)将会造成损失

## 构建打包

本项目使用 GitHub Actions 自动构建。构建流程：

1. 从 [Grafana Loki Releases](https://github.com/grafana/loki/releases) 下载对应架构的二进制文件
2. 替换`package/manifest`版本号和架构信息
3. 打包为 fnOS 应用格式（`.tgz`），你喜欢的话修改扩展名为`fpk`也行

### 手动触发构建

1. 进入 GitHub Actions 页面
2. 选择 "Test Release" 工作流
3. 点击 "Run workflow"
4. 选择是否创建 Release，如果创建了release就会在release出现

### 版本更新
一般来说启动脚本或者是打包流程没变的话  
修改 `.github/workflows/main.yml` 中的版本号：
```yaml
env:
  target_version: "3.6.7"              # Loki版本
  this_pack_manifest_version: "3.6.7-1" # 打包的安装包版本，后面那个-1主要是在主版本不变时有改动修改
```  
就能打出新包了
## 相关链接

- [Grafana Loki 官方文档](https://grafana.com/docs/loki/latest/)
- [Grafana Loki GitHub](https://github.com/grafana/loki)
- [飞牛NAS 官网](https://www.fnnas.com/)
