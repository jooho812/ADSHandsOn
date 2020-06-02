## Azure Cosmos DB

### 01. Azure Cosmos DB MongoDB API
```powershell
$resourceGroup="rg-aztest"
$location="koreacentral"
$accountName="myAccountName"

az cosmosdb create -n $accountName -g $resourceGroup --kind MongoDB --default-consistency-level Eventual --locations regionName=$location
```


robo 3t를 실행 하여 mongodb query로 데이터베이스 및 컬렉션을 생성 합니다  

```c#
// azcosmostest 라는 데이터베이스를 생성 합니다  
use azcosmostest
db.runCommand({customAction: "CreateDatabase"});

// restaurants 라는 컬렉션을 10000 RU 로 생성 합니다  
use azcosmostest
db.runCommand({customAction: "CreateCollection", collection: "restaurants", offerThroughput: 10000});
// 생성된 컬렉션 확인
db.runCommand({customAction:'getCollection',collection:'restaurants'})
```

테스트 파일을 다운로드 및 cosmosdb에 삽입 합니다  
```powershell
[Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12
Invoke-WebRequest -Uri "https://raw.githubusercontent.com/mongodb/docs-assets/primer-dataset/primer-dataset.json" -OutFile C:\restaurants.json; 

# mongoimport를 다운로드 합니다
Invoke-WebRequest -Uri "https://azmyhanson.blob.core.windows.net/azcon/mongoimport.exe" -OutFile c:\mongoimport.exe; 

c:\mongoimport -h {cosmoshost}.documents.azure.com:10255 -d azcosmostest -c restaurants -u azsycosmos -p {password} --ssl --file c:\restaurants.json
```



이제 몇 가지의 쿼리를 테스트 해볼 수 있습니다  

```c#
// name field가 존재하고 field값이 공백이 아닌 document를 name field 기준 오름차순으로 조회
db.restaurants.find({name:{"$exists":true, $ne:""}}).sort({name:1})

// 특정 쿼리의 소비 RU를 확인하고 싶다면 쿼리 이후 아래의 커맨드로 소모한 RU를 확인할 수 있습니다
db.runCommand({'getLastRequestStatistics':1})

// cuisine field값이 'American' 인 document를 name field 기준 내림차순으로 조회
db.restaurants.find({cuisine:"American"}).sort({name:1})
db.runCommand({'getLastRequestStatistics':1})

// cuisine field값에 'Bagels' 문자열이 포함된 document를 name field 기준 내림차순으로 조회
db.restaurants.find({cuisine:/Bagels/}).sort({name:1})
db.runCommand({'getLastRequestStatistics':1})

// borough field값에 대한 중복제거
db.restaurants.distinct("borough")
db.runCommand({'getLastRequestStatistics':1})

// address document field에 포함된 child document field 값이 일치하는 document를 조회
db.restaurants.find({"address.zipcode":"11224", "address.building":"2780"})
db.runCommand({'getLastRequestStatistics':1})

// grades document field에 포함된 child array에 포함된 document의 grade field값중 'A' 를 포함하는 document를 조회
db.restaurants.find({"grades.grade":"A"})
db.runCommand({'getLastRequestStatistics':1})

//위 쿼리 결과 (grades.grade = A) cuisine 별로 카운트, 많은 수대로 정렬
db.restaurants.aggregate([
    { $match: {"grades.grade":"A"}},
    { $group: { _id: "$cuisine", total: { $sum: 1 } } },
    { $sort: { total: -1 } }
])
db.runCommand({'getLastRequestStatistics':1})

```

mongodb에서 지원하는 몇몇 기능들은 사용할 수 없는 기능이 있습니다  
집계 파이프라인과 유선 프로토콜은 public preview로 제공 됩니다  

azure portal에서 cosmos db account를 선택 합니다  
왼쪽 메뉴에서 미리 보기 기능을 선택 합니다  
집계 파이프라인과 MongoDB 3.4 유선 프로토콜을 사용으로 변경 합니다  

완료 된 이후 동일 쿼리를 다시 실행해 봅니다  


### 02. Azure CosmosDB SQL API
```powershell
$resourceGroup="rg-aztest"
$location="koreacentral"
$accountName="myAccountName"

az cosmosdb create -n $accountName -g $resourceGroup --default-consistency-level Eventual `
--locations regionName=koreacentral failoverpriority=0 `
--locations regionName=koreasouth failoverpriority=1
```
https://raw.githubusercontent.com/Azure-Samples/azure-cosmos-db-sample-data/master/SampleData/VolcanoData.json


### Multi master Geo-Replication


### Index?