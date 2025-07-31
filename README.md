# [내일배움캠프 사전캠프] 쉽게 배우는 C++ 언리얼 엔진 3D 게임 개발 기초 5주차 및 마무리

## 1. 오늘 학습 키워드

>## 게임모드에서 시간마다 스테이지가 진행되면서 정해전 적 AI 소환

>## 기존과 공격 방식이 다른 적 AI를 생성하고, 소환할때 2가지 AI 중 랜덤으로 선택

## 2. 오늘 학습 한 내용을 나만의 언어로 정리하기

### 1. 게임모드에서 시간마다 스테이지가 진행되면서 정해전 적 AI 소환

**MainGameMode.h**

```cpp
public:

	float Time;
```


**MainGameMode.cpp**

```cpp
void AMainGameMode::BeginPlay()
{
	Time = 0;
  Stage = 0;
}

	Time += DeltaSeconds;
	UWorld* World = GetWorld();
	if (!IsValid(World))
	{
		return;
	}
	if (!IsValid(Enemy))
	{
		return;
	}

	if ((int)(Time / 30 + 1) != Stage)
	{
		Stage++;

		for (int i = 0; i < Stage; i++)
		{
			UNavigationSystemV1* NavSystem = UNavigationSystemV1::GetNavigationSystem(GetWorld());
			FNavLocation LOC;
			NavSystem->GetRandomPoint(LOC);
			FActorSpawnParameters SpawnParams;
			SpawnParams.SpawnCollisionHandlingOverride = ESpawnActorCollisionHandlingMethod::AlwaysSpawn;

			GetWorld()->SpawnActor<AActor>(Enemy, LOC.Location, FRotator::ZeroRotator, SpawnParams);
		}
	}

```

기존 **MainGameMode**헤더에 시간과 스테이지를 저장하는 변수를 지정하고,

게임이 시작될 때 시간과 스테이지를 0으로 초기화하고 Time 변수에 틱마다 시간이 추가되게 구현하였다.

Time을 특정 시간마다 스테이지를 증가할 수 있게 if문을 작성하였고

이 스테이지가 증가할 때 마다 스테이지의 숫자만큼 적이 생성되게 코드를 작성하였다.

### 2. 기존과 공격 방식이 다른 적 AI를 생성하고, 소환할때 2가지 AI 중 랜덤으로 선택

**AIBoxBase.h**

```cpp
#pragma once

#include "CoreMinimal.h"
#include "CharacterBase.h"
#include "AIBoxBase.generated.h"

/**
 * 
 */
UCLASS()
class BASIS_API AAIBoxBase : public ACharacterBase
{
	GENERATED_BODY()
	
	virtual void Attack() override;
	virtual void Hit(int32 Damage, AActor* ByWho) override;
};


```

**AIBoxBase.cpp**

```cpp


#include "AIBoxBase.h"
#include "Engine/OverlapResult.h"

void AAIBoxBase::Attack()
{
	Super::Attack();
	TArray<FOverlapResult> result;
	GetWorld()->OverlapMultiByChannel(result, GetActorLocation() + GetActorForwardVector() * 100, FQuat::Identity, ECollisionChannel::ECC_Camera, FCollisionShape::MakeSphere(1000));
	for (int i = 0; i < result.Num(); i++)
	{
		ACharacterBase* Character = Cast<ACharacterBase>(result[i].GetActor());
		if (IsValid(Character) && Character !=this)
		{
			Character->Hit(Strength, this);
			break;
		}

	}
}

void AAIBoxBase::Hit(int32 Damage, AActor* ByWho)
{
	Super::Hit(Damage, ByWho);

	if (CurrentHP > 0)
	{
		return;
	}
	if (CurrentHP <= 0)
	{
		ACharacterBase* CB = Cast<ACharacterBase>(ByWho);
		if (IsValid(CB))
		{
			CB->IncreaseKillCount();
		}
	}

	Destroy();
}

```

기존 **CharacterBase를 부모**로 삼고, **AIPlayer에서 근접 공격에 필요한 부분들만 코드**를 가져와 재구성했다.

근접 공격은 기존에 **AI가 플레이어를 찾는** 코드에서 범위를 줄이고 **범위안에 들어오면 공격을 진행**하는 방식으로 구현하였다.

또한 기존에 만든 AI와 랜덤하게 스폰을 정하기 위해 **MainGameBase**를 추가로 수정하였다.

**.h**

```cpp

	UPROPERTY(EditAnywhere)
	TSubclassOf<AActor> Enemy2;

```

**.cpp**

```cpp
	if (!IsValid(Enemy))
	{
		return;
	}
	if (!IsValid(Enemy2))
	{
		return;
	}
	if ((int)(Time / 30 + 1) != Stage)
	{
		Stage++;

		for (int i = 0; i < Stage; i++)
		{
			UNavigationSystemV1* NavSystem = UNavigationSystemV1::GetNavigationSystem(GetWorld());
			FNavLocation LOC;
			NavSystem->GetRandomPoint(LOC);
			FActorSpawnParameters SpawnParams;
			SpawnParams.SpawnCollisionHandlingOverride = ESpawnActorCollisionHandlingMethod::AlwaysSpawn;

			TSubclassOf<AActor> summon = nullptr;

			if (FMath::Rand() % 2 == 0)
			{
				summon = Enemy;
			}

			else 
			{
				summon = Enemy2;
			}
			GetWorld()->SpawnActor<AActor>(summon, LOC.Location, FRotator::ZeroRotator, SpawnParams);
		}
	}
```

summon을 추가 변수로 두고 랜덤하게 서몬에 적 AI들을 대입하고, 기존에 Enemy를 소환하던 코드를 summon 바꾸어서 AI 두 종류가 스테이지가 올라갈 때 마다 스폰되게 만들었습니다.

#내일배움캠프 #사전캠프 #TIL 
 
