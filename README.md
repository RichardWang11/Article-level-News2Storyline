We wish to organized the news article to structured storyline.
We do a lot of tasks about mining the semantic of raw text, and try to figure out what relations between this sentences.
Here are same of examples:
| id | topic | task | source | url |
|----|-------|------|--------|-----|
| 0 |新冠疫情| 新冠疫情文本清洗和去重| 新闻网站|——|
| 1 |新冠疫情|政策与新冠疫情事件语义相似度|新闻网站|——|
| 2 |新冠疫情|政策与新冠疫情事件语义搜索|新闻网站|——|
| 3 |新冠疫情|新冠疫情文本语义角色标注|新闻网站&LTP|——|
| 4 |新冠疫情|新冠疫情新闻故事脉络生成|新闻网站&BERT&聚类|——|
第一阶段，文本清洗和去重，其中去重部分主要包括新闻内容重复（计算文本的相似度）。
第二阶段，政府发行的新冠政策（如二十条），与爬取到的新冠疫情主题事件之间的语义相似度。
第三阶段，政府发行的新冠政策（如二十条），每一条对应的最相关的新冠疫情事件（Top5）。
第四阶段，将新冠疫情主题事件通过语义角色标注的方法，获取每一条事件的属性，主要包括施事者(causer)、受事者(patient)和谓语动词(predicate)。
第五阶段，梳理和重构2022年内发生的与新冠疫情主题相关的事件，整理出一条故事脉络，更清晰地看出这些事件间的关联。
例子：
第一阶段：
# 读取csv文件
df = pd.read_csv('./z疫情csv/7SH_cleaned.csv')

# 将"result"列转换为文本矩阵
vectorizer = CountVectorizer()
X = vectorizer.fit_transform(df['result'])

# 计算相似度矩阵
similarity_matrix = cosine_similarity(X)

# 获取相似度大于0.8的新闻的下标
indices = pd.Series(df.index)
duplicates = []
for i in range(len(similarity_matrix)):
    if i not in duplicates:
        for j in range(len(similarity_matrix)):
            if j not in duplicates and i != j:
                if similarity_matrix[i][j] > 0.8:
                    duplicates.append(j)

# 删除相似度大于0.8的新闻
df.drop(df.index[duplicates], inplace=True)

# 保存结果到新的csv文件
df.to_csv('./z疫情csv/7SH_Ccleaned.csv', index=False)

# 输出提示信息
print('按照新闻内容相似度去重后的数据已保存到new_data.csv文件中')
第二阶段：
将政策句子和新闻事件句子都使用sentence_bert进行编码，然后输出两者的语义相似度（cos_similarity）。
输出的结果：
| id | topic | task | policy | news_event |
|----|-------|------|--------|-----|
| 0 |新冠疫情| 政策与新冠疫情语义相似度| 二是进一步优化核酸检测。不按行政区域开展全员核酸检测，进一步缩小核酸检测范围、减少频次。根据防疫工作需要，可开展抗原检测。对高风险岗位从业人员和高风险区人员按照有关规定进行核酸检测，其他人员愿检尽检。除养老院、福利院、医疗机构、托幼机构、中小学等特殊场所外，不要求提供核酸检测阴性证明，不查验健康码。重要机关、大型企业及一些特定场所可由属地自行确定防控措施。不再对跨地区流动人员查验核酸检测阴性证明和健康码，不再开展落地检。 |优化落实疫情防控“新十条”发布春节前夕机票搜索量猛增12月7日，国务院联防联控机制发布《关于进一步优化落实新冠肺炎疫情防控措施的通知》，包括不再对跨地区流动人员查验核酸检测阴性证明和健康码，不再开展落地检等新措施。业内人士称，消息发布后，携程平台上的机票瞬时搜索量猛增160，其中，春节前夕的机票搜索量暴涨至三年以来最高点。|
