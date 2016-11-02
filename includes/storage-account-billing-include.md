我们将根据你的存储帐户，针对你的 Azure 存储空间使用情况收费。存储成本取决于以下几个因素：区域/位置、帐户类型、存储容量、复制方案、存储交易和数据流出量。

- 区域是指你的帐户所在的地理区域。
- 帐户类型是指你使用的是通用存储帐户还是 Blob 存储帐户。如果使用的是 Blob 存储帐户，则访问层还可以确定该帐户的计费模型。
- 存储容量指的是存储帐户中用来存储数据的配额。
- 复制可以确定一次保留的数据副本的数量以及保留位置。
- 事务指的是对 Azure 存储空间的所有读取和写入操作。
- 数据流出量指的是传出某个 Azure 区域的数据。当不在同一区域中的应用程序访问你的存储帐户中的数据时，你需要为数据流出量付费。（对于 Azure 服务，你可以采取措施将你的数据和服务通过分组分到相同的数据中心内，从而降低或避免数据流出量费用。）

[Azure 存储定价](https://azure.microsoft.com/pricing/details/storage/)页提供基于帐户类型、存储容量、复制和交易的详细定价信息。[数据传输定价详细信息](https://azure.microsoft.com/pricing/details/data-transfers/)提供了针对数据流出量的详细定价信息。你可以使用 [Azure 存储空间定价计算器](https://azure.microsoft.com/pricing/calculator/?scenario=data-management)来帮助估算成本。

<!---HONumber=AcomDC_0921_2016-->