# serverless
본 문서에서는 AWS Lambda, Amazon API Gateway, Amazon DynamoDB, Amazon CloudFront를 이용해서 CRUD가 가능한 간단한 REST API를 설계하는 것을 목표로 합니다.

각각의 기능이 어떤 역할을 하는지는 본 문서에서 차례대로 설명하도록 하겠습니다. 대략적인 다이어그램은 아래와 같습니다.

![serverless diagram](../images/serverless-diagram.png)

본 워스샵의 아키텍처는 간단합니다. User는 CloudFront에 연결된 API GateWay에 요청을 보냅니다. 요청을 받은 API GateWay는 트리거로 설정된 Lambda를 실행하고  DaynomoDB에 데이터를 Create, Readt, Update, Delete를 합니다.

### 리전 선택
실습에 앞서 AWS 콘솔의 우측 상단의 리전 드롭다운 메뉴에서 다음 서비스를 지원하는 리전을 선택합니다.

(Seoul 리전은 위 서비스를 모두 지원합니다.)

- AWS Lambda
- Amazon API Gateway
- Amazon DynamoDB
- Amazon CloudFront

![serverless select regeion](../images/serverless-select-regeion.png)

## Amazon DynamoDB

> Amazon DynamoDB는 AWS에서 NoSQL 데이터베이스입니다. 또한 완전 관리형 클라우드 데이터베이스로 안정적인 성능을 기대할 수 있습니다. 자세한 설명은 [아마존 공식 문서](https://aws.amazon.com/dynamodb)를 참고해주세요.

첫 번째로 설정해야 하는 부분은 Amazon DynamoDB입니다. 실습의 간단함을 위해 "id", "text" 두 개의 키를 가지는 "Comment" 테이블을 만들어 보겠습니다.

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
5. key값은 "text"를 입력하고 내용는 "hello dynamodb"를 입력합니다.

![serverless enter item dynamodb](../images/serverless-enter-item-dynamodb.png)

6. 저장을 클릭합니다.

새롭게 항목이 추가된 것을 확인할 수 있습니다. 동일한 방법으로 2~3개의 아이템을 더 추가해 보세요.
