---
layout: post
title: "sequelize 에서 사전에 작성된 테이블 사용시 모델 작성방법"
tags: [node, orm, sequelize]
comments: true
---

sequelize는 node.js에서 많이 사용되는 ORM 라이브러리다. 다양한 종류의 데이터베이스를 지원하고, 기능과 문서화 및 사용자 커뮤니티 규모 면에서도 여러모로 부족함이 없는 라이브러리이다. 그런데 기존에 작성된 데이터베이스와 테이블을 이용하고자 할 경우 모델 작성시 몇가지 고려해야 할 사항이 있다.

# 문제의 시작

node.js로 외부 장비에서 로그메세지를 수신해서 MariaDB에 저장하는 로그 서비스를 작성한다고 하자. 단순 로그메세지 이므로 기본키 없이 장비구분과 로그메세지 본문, 그리고 로그 생성시간만을 기록하는 테이블에 메세지를 기록만 하고자 하면 다음과 같이 가장 단순한 형태의 모델을 작성 할 수 있을 것이다.

```javascript
module.exports = (sequelize, DataTypes) => { 
    // Model definition
    let LogModel = sequelize.define('StatusLog', { 
        device_id: { type: DataTypes.STRING(100), allowNull: false },
        log: { type: DataTypes.TEXT, allowNull: true }, 
        create_time: { type: DataTypes.DATE, allowNull: false }
    });

    return LogModel;
};
```

그런데 이 모델을 사용하여 테이블에 로그메세지를 저장하려 시도하는 경우 다음 3가지 문제가 발생할 수 있다.

* ```id``` 컬럼이 없다는 오류 발생
* ```createdAt``` 과 ```updatedAt``` 컬럼이 없다는 오류 발생
* 테이블 이름의 기본값이 ```모델이름s```로 자동으로 복수형 테이블명을 사용하는 문제

# 문제 해결

### _id 컬럼이 없다는문제

sequelize는 1개의 primary key를 필요로 한다. primary key가 없는 경우 id 컬럼을 암시적으로 생성해서 primary key로 지정한다.

따라서 다음 두 가지 해결방법이 있다

* primary key 에 해당하는 컬럼을 생성하는 방법
* 모델을 생성한 다음 자동으로 추가된 id 컬럼을 삭제하는 방법

여기에서는 기존에 생성된 테이블을 사용해야 하므로 모델객체 생성 후 id 컬럼을 삭제하는 방법을 고려 할 것이다.

### createdAt 과 updatedAt 컬럼이 없다는 오류 발생

sequelize 는 데이터 생성시와 수정시 시간을 기록하는 createdAt 컬럼과 updatedAt 컬럼을 기본적으로 생성하고 사용한다.

이 컬럼은 모델 생성시 ```timestamp``` 옵션으로 사용유무를 결정 할 수 있으며, 사용하고자 할 경우 컬럼 이름을 사용자가 임의로 지정해 줄 수 있는 기능을 지원한다.

### 수형 테이블명을 사용하는 문제

sequelize 는 테이블 이름을 기본적으로 복수형으로 생성한다.

테이블 이름이 바뀌는 것을 원하지 않는 경우 ```freezeTableName```옵션을 주고, ```tableName```옵션에 사용하고자 하는 테이블 이름을 명기해 주면 된다.


# 수정된 모델 예시

그래서 위 내용을 종합하면 다음과 같이 모델을 작성 할 수 있을 것이다.

```javascript
module.exports = (sequelize, DataTypes) => { 
    // Model definition
    let LogModel = sequelize.define('StatusLog', { 
        device_id: { type: DataTypes.STRING(100), allowNull: false },
        log: { type: DataTypes.TEXT, allowNull: true }, 
        create_time: { type: DataTypes.DATE, allowNull: false }
    },
    // Model options 
    {
        timestamps: false,
        freezeTableName: true,
        tableName : "TB_LOG"
    });
    
    // Remove default primary Keys
    LogModel.removeAttribute('id');

    return LogModel;
};
```

# 참고 링크

* https://sequelize.org/master/manual/models-definition.html
* https://stackoverflow.com/questions/29233896/sequelize-table-without-column-id
* https://stackoverflow.com/questions/36906500/avoid-created-at-and-updated-at-being-auto-generated-by-sequelize
* https://stackoverflow.com/questions/21114499/how-to-make-sequelize-use-singular-table-names/34558832