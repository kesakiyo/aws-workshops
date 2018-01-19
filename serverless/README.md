# serverless
본 문서에서는 AWS Lambda, Amazon API Gateway, Amazon DynamoDB를 이용해서 CRUD가 가능한 간단한 REST API를 설계하는 것을 목표로 합니다.

각각의 기능이 어떤 역할을 하는지는 본 문서에서 차례대로 설명하도록 하겠습니다. 대략적인 다이어그램은 아래와 같습니다.

![serverless diagram](../images/serverless-diagram.png)

본 워스샵의 아키텍처는 간단합니다. User는 API GateWay에 요청을 보냅니다. 요청을 받은 API GateWay는 트리거로 설정된 Lambda를 실행하고  DaynomoDB에 데이터를 Create, Readt, Update, Delete를 합니다.

### 리전 선택
실습에 앞서 AWS 콘솔의 우측 상단의 리전 드롭다운 메뉴에서 다음 서비스를 지원하는 리전을 선택합니다.

(본 워크샵에서는 다음 서비스를 모두 지원하는 Seoul 리전을 사용한다고 가정합니다.)

- AWS Lambda
- Amazon API Gateway
- Amazon DynamoDB
- Amazon CloudFront

![serverless select regeion](../images/serverless-select-regeion.png)

## Amazon DynamoDB

> Amazon DynamoDB는 AWS에서 NoSQL 데이터베이스입니다. 또한 완전 관리형 클라우드 데이터베이스로 안정적인 성능을 기대할 수 있습니다. 자세한 설명은 [아마존 공식 문서](https://aws.amazon.com/dynamodb)를 참고해주세요.

첫 번째로 설정해야 하는 부분은 Amazon DynamoDB입니다. 간단한 실습을 위해 "id", "text" 두 개의 키를 가지는 "Comment" 테이블을 만들어 보겠습니다.

### DynamoDB 테이블 생성
1. AWS 콘솔에서 Amazon DynamoDB를 선택합니다.
2. "테이블 만들기"를 선택합니다.
3. 테이블 이름은 "Comment", 기본 키는 "id"를 입력합니다. 그 외의 값은 기본값으로 설정합니다.

![serverless create dynamodb](../images/serverless-create-dynamodb.png)

4. 생성을 클릭합니다.

테이블 생성이 완료되면 활성 상태가 된 것을 확인합니다. 활성 상태가 되기까지 보통 수 초 이내에 완료됩니다. 활성 상태가 되면 아래 그림과 같이 초록색 글씨로 "활성"이 표시 됩니다.

![serverless create dynamodb](../images/serverless-active-dynamodb.png)

CRUD를 할 수 있는 테이블이 생성됐습니다!! 이제 수동으로 데이터를 추가해 보겠습니다.

### 수동으로 데이터 추가
1. Comment 테이블을 클릭합니다.
2. 상세 보기에서 "항목" - "항목 만들기" 차례로 클릭합니다.

![serverless create item dynamodb](../images/serverless-create-item-dynamodb.png)

3. id는 1을 입력합니다.
4. 왼쪽에 있는 "+"를 클릭한 뒤 "String"을 선택합니다.
5. key값은 "content"를 입력하고 내용는 "hello dynamodb"를 입력합니다.

![serverless enter item dynamodb](../images/serverless-enter-item-dynamodb.png)

6. 저장을 클릭합니다.

새롭게 항목이 추가된 것을 확인할 수 있습니다. 동일한 방법으로 2~3개의 아이템을 더 추가해 보세요.

## AWS Lambda
> AWS는 서버를 프로비저닝하거나 관리할 필요 없이 코드 조각을 실행하는 서비스입니다. Lambda를 실행하기 위해서는 트리거를 설정해 줘야하는데 DynamoDB, API GateWay등 다양한 서비스들과의 인테그레이션을 제공합니다. 자세한 내용은 [아마존 공식 문서](https://aws.amazon.com/lambda)를 참고해 주세요.

다음은 본 문서의 핵심 기능인 AWS Lambda 함수를 만드는 과정입니다. 이 순서에서는 총 네가지의 함수를 만듭니다. 각각의 함수는 Create, Read, Update, Delete 기능을 하게 됩니다. Lambda 함수를 만들기 전에 함수가 AWS 자원들에 접근할 수 있는 역할을 만들어 줘야 합니다.

### IAM 역할 생성
1. AWS 콘솔에서 IAM을 선택합니다.
2. "역할" - "역할 만들기"를 차례로 클릭합니다.

![serverless create iam](../images/serverless-create-iam.png)

3. "AWS 서비스" - "Lambda"를 선택한 뒤 "다음"을 클릭합니다.

![serverless select lambda iam](../images/serverless-select-lambda-iam.png)

4. AWSLambdaBasicExecutionRole, AmazonDynamoDBFullAccess를 검색해 선택합니다.
5. 역할 이름은 lambda-iam 이라고 입력합니다. **정책 리스트에 4번에서 선택한 두 개가 있는지 확인합니다.**

![serverless confirm iam](../images/serverless-confirm-iam.png)

6. 역할 만들기를 클릭합니다. 역할 리스트에서 lambda-iam 역할이 만들어진것을 확인합니다.

### Lambda 함수 생성

성공적으로 IAM 역할을 만들었으면 DynamoDB를 조작하는 Lambda 함수를 만들 차례입니다.

1. AWS 콘솔에서 AWS Lambda를 선택합니다.
2. 함수 만들기를 클릭합니다.
3. 함수 이름은 create-comment, 런타임은 Node.js 6.10, 역할은 전 순서에서 만들었던 lambda-iam을 선택합니다.

![serverless create lambda](../images/serverless-create-lambda.png)

4. 함수 생성을 클릭합니다.

### Lambda 함수 코드 입력
가장 기본적인 Lambda 함수를 만들었습니다!! 이제 여러분은 이 Lambda 함수를 이용해 DynamoDB에 FullAccess를 할 수 있습니다. 이제 이 Lambda 함수를 이용해 Comment를 생성하는 기능을 구현해 보겠습니다. 

스크롤을 내이다 보면 중간쯤에 "함수 코드"라고 적혀져 있는 온라인 에디터를 볼 수 있습니다. 에디터에 아래 코드를 입력한 뒤 **우측 상단에 있는 "저장" 버튼을 클릭해주세요**

![serverless create code lambda](../images/serverless-create-code-lambda.png)

```javascript
// Create Comment
const AWS = require('aws-sdk');
const documentClient = new AWS.DynamoDB.DocumentClient();
const uuid = require('uuid/v4');

exports.handler = (event, context, callback) => {
  const params = {
    Item : {
      id: uuid(),
      content: event.content,
    },
    TableName : 'Comment',
  };

  documentClient.put(params, (err, data) => {
    if (err) {
      return callback(err, null);
    }
  
    callback(null, data.Item);
  });
};
```

> 간단한 실습을 위해 uuid 라이브러리를 사용해 id를 생성합니다. 실제 제품에서는 uuid를 id로 사용하지는 않습니다.

### Lambda 테스트 작성
코드 저장이 완료됐으면 우리가 생성한 Lambda function이 잘 동학하는지 테스트를 해 볼 차례입니다.

1. 우측 상단에 있는 테스트 버튼을 클릭합니다.
2. 적당한 테스트 이름을 입력하고 아래와 같이 테스트에 사용할 JSON 내용을 입력합니다.

![serverless create test lambda](../images/serverless-create-test-lambda.png)

```javascript
{
  "content": "My first Lambda function!!"
}
```

3. "생성" 버튼을 클릭합니다.
4. 테스트 생성이 완료되면 우측 상단에 있는 "테스트" 버튼을 눌러 테스트가 성공적으로 돌아가는지 확인합니다.
5. 아래 그림과 같이 "성공" 키워드와 함께 토스트가 보인다면 Lambda 함수가 성공적으로 생성된 것입니다.

![serverless success test lambda](../images/serverless-success-test-lambda.png)

6. DaynamoDB Comment 테이블에 새로운 항목이 생성이 된 것을 확인합니다.

### Lambda 나머지 함수 구현
성공적으로 Create Lambda 함수를 구현했으면 나머지 Read, Update, Delete 함수도 구현합니다. 함수 이름은 아래와 같습니다. 다음 순서인 Amazon API Gateway와 연결을 해줘야 하기 때문에 **함수 이름은 서로 구분이 되어야 합니다.**
- Create Lambda 함수 이름: create-comment
- Read Lambda 함수 이름: read-comment
- Update Lambda 함수 이름: update-comment
- Delete Lambda 함수 이름: delete-comment

**나머지 Lambda 함수를 만드는 과정은 Create Lambda 함수를 만드는 과정과 정확히 일치합니다.** 차이점이 있다면 단지 코드뿐입니다. 아래는 나머니 세 기능에 대한 코드입니다.

#### Read Lambda 함수 코드
```javascript
// Read Comments
const AWS = require('aws-sdk');
const documentClient = new AWS.DynamoDB.DocumentClient();
const uuid = require('uuid/v4');

exports.handler = (event, context, callback) => {
  const params = {
    TableName : 'Comment',
  };
  documentClient.scan(params, (err, data) => {
    if(err) {
      return callback(err, null);
    }
    callback(null, data.Items);
  });
};
```

#### Update Lambda 함수 코드
```javascript
// Update Comment
const AWS = require('aws-sdk');
const documentClient = new AWS.DynamoDB.DocumentClient();
const uuid = require('uuid/v4');

exports.handler = (event, context, callback) => {
  const params = {
    UpdateExpression: 'set content=:c',
    ExpressionAttributeValues:{
      ':c': event.content,
    },
    ReturnValues:"UPDATED_NEW",
    Key: {
      id: event.id,
    },
    TableName : 'Comment',
  };

  documentClient.update(params, (err, data) => {
    if (err) {
      return callback(err, null);
    }
    callback(null, data);
  });
};
```

#### Delete Lambda 함수 코드
```javascript
// Delete Comment
const AWS = require('aws-sdk');
const documentClient = new AWS.DynamoDB.DocumentClient();
const uuid = require('uuid/v4');

exports.handler = (event, context, callback) => {
  const params = {
    Key: {
      id: event.id,
    },
    TableName : 'Comment',
  };
	
  documentClient.delete(params, (err, data) => {
    if (err) {
      return callback(err, null);
    }

    callback(null, data);
  });
};
```

> Read, Update, Delete Lambda 함수를 성공적으로 작성했다면 각각의 함수에 대해 테스트 케이스를 만들어 테스트를 해 봅니다. 코드를 잘못 입력해 Lambda 함수가 동작하지 않는 버그를 손쉽게 찾을 수 있습니다.

## Amazon API Gateway
> Amazon API Gateway는 누구든지 손쉽게 API를 생성, 관리 할 수 있게 해주는 서비스입니다. AWS Lambda에서 실행되는 코드, 또는 백엔드 서비스에 액세스 할 수 있게 해주는 역할을 합니다. 자세한 내용은 [아마존 공식 문서](https://aws.amazon.com/api-gateway)에서 확인하세요.


### API Method 생성
우리는 위 순서에서 CRUD가 가능한 AWS Lambda 함수들을 만들었습니다. 하지만 외부에서 AWS Lambda 함수를 실행시킬 방법이 현재 없습니다. 함수를 실행시키기 위해서는 어떠한 트리거가 있어야 하는데 본 워크샵에서는 그 역할을 Amazon API Gateway가 합니다.

1. 새 API를 만드는 화면에 엑세스합니다. 만약 생성된 API가 하나도 없다면 "시작" 버튼을 클릭합니다.
2. "새 API"를 선택하고 API 이름은 lambda로 입력합니다. 적당한 설명을 입력하고 그 외의 설정은 기본값을 유지합니다.

![serverless create api](../images/serverless-create-api.png)

3. "API 생성" 버튼을 클릭합니다.
4. 상단에 있는 "작업" 버튼을 선택하고 "메서트 생성" 을 클릭합니다.

![serverless select method](../images/serverless-select-method.png)

5. "GET" 메서드를 선택하고 생성을 완료합니다.

![serverless select get method](../images/serverless-select-get-method.png)

6. 생성된 "GET" 메서드를 클릭해 통합유형, Lambda 리전, Lambda 함수 이름을 설정해줍니다. 그 외 설정은 기본값을 유지합니다.
- 통합유형: Lambda 함수
- Lambda 리전: ap-northeast-2
- Lambda 함수: read-comment (혹 Read Comment Lambda 함수 이름을 문서와 다르게 설정했다면 그 이름을 입력해줍니다.)

![serverless select get method](../images/serverless-set-get-method.png)

7. 설정을 눌러 설정을 저장합니다. 추가 권한이 필요하다는 알림창이 나오면 확인을 누릅니다.

"GET" 메서드 생성과 설정이 완료됐다면 이제 Read Lambda 함수를 실행할 준비가 됐습니다. 동일한 방법으로 **POST, PUT, DELETE** 메서드를 추가합니다. 다른 메스드를 추가하는 방법은 "GET" 메서드를 추가하는 방법과 거의 동일합니다. 

단 하나 차이점이 있다면 메서드가 실행시킬 Lambda 함수 이름입니다. 각각의 메서드와 연결될 Lambda 함수 이름은 아래와 같습니다.

- GET: read-comment
- POST: create-comment
- PUT: update-comment
- DELETE: update-comment

### API 배포
GET, POST, PUT, DELETE API를 생성했다면 이제 배포를 해 접근 가능한 주소를 획득해야 합니다.

1. 상단에 있는 "작업" - "API 배포" 순으로 클릭합니다.
2. 배포 스테이지에서 [새 스테이지]를 선택합니다.
3. 스테이지 이름에 "prod"로 입력합니다.

![serverless enter stage deploy](../images/serverless-enter-stage-deploy.png)

4. "배포" 버튼을 클릭합니다.

배포가 완료되면 아래와 같이 배포된 API Gateway에 접근할 수 있는 주소를 확인할 수 있습니다. 주소를 클릭해 GET요청으로 Amazon DynamoDB에 있는 Comment 데이터들이 정상적으로 불러와지는지 확인해보세요.

![serverless enter stage deploy](../images/serverless-complete-deploy.png)

## API Test
이제 여러분은 [id, text] 키를 가진 Comment 테이블에 CRUD를 하는 RestAPI 서버를 가지고 있습니다👏👏👏 

남은 일은 여러분이 가진 REST API를 테스트 할 일입니다. REST API를 테스트 할 수 있는 방법은 여러가지가 있지만 가장 간단한 방법은 대중적으로 알려진 크롬 확장프로그램인 [Post man](https://chrome.google.com/webstore/detail/postman/fhbjgbiflinjbdggehcddcbncdddomop)을 사용하는 방법입니다.

혹은 본 문서에 포함된 CommentTester.zip 파일을 받아 압축을 푼 뒤 index.html을 실행해 여러분의 REST API가 제대로 동작을 하는지 확인해보세요.
> 압축파일에 있는 index.html 파일과 bundle.js 파일이 한 폴더에 있어야 함을 주의하세요.
