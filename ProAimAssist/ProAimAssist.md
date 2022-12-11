# ProAimAssist

## 一、功能列表

- <font color='lightgreen'>**灵敏度修正**</font>
- <font color='lightgreen'>**磁性子弹**</font>
- <font color='lightgreen'>**辅助瞄准范围修正**</font>
- <font color='lightgreen'>**自动瞄准**</font>

<details>
<summary><font color='lightblue'>代码</font></summary>

```c++
void UMyAimAssistComponenet::TickComponent(float DeltaTime, ELevelTick TickType, FActorComponentTickFunction* ThisTickFunction)
{
	Super::TickComponent(DeltaTime, TickType, ThisTickFunction);
	if (AimAssistDataAsset == nullptr || !PlayerController.IsValid() || PlayerController->GetPawn() == nullptr)
	{
		return;
	}
	HandleCurrentTarget();               //处理各个功能是否开启
	UpdateCrosshair(DeltaTime);          //磁性子弹
	UpdateZonesScaling(DeltaTime);       //辅助瞄准范围修正
	HandleAutoAim(DeltaTime);            //自动瞄准
#if !UE_BUILD_SHIPPING
	DrawDebug(DeltaTime);
#endif
}
```
</details>


## 二、配置文件
#### <font color='red'>UASAimAssistConfigDataAsset</font>

<details>
<summary>AimAssistConfig(通用配置UUASAimAssistConfigDataAsset）</summary>
<font color='orange'>UpdateTargetsRate</font><font color='aqua'>(float)</font>:更新目标频率<br>
<font color='orange'>AimAreaExtents</font><font color='aqua'>(Fvector)</font>:检测范围<br>
<font color='orange'>bStickinessZoneConfig</font><font color='aqua'>(bool)</font>:是否开启<font color='lightgreen'>灵敏度修正</font><br>
<font color='orange'>StickinessZoneConfig</font><font color='aqua'>(FUASStickinessZoneConfig)</font>:<font color='lightgreen'>灵敏度修正</font>配置项<br>
<font color='orange'>bMagnetismZoneConfig</font><font color='aqua'>(bool)</font>:是否开启<font color='lightgreen'>磁性子弹</font><br>
<font color='orange'>MagnetismZoneConfig</font><font color='aqua'>(FUASMagnetismZoneConfig)</font>:<font color='lightgreen'>磁性子弹</font>配置项<br>
<font color='orange'>bScalingZoneConfig</font><font color='aqua'>(bool)</font>:是否开启<font color='lightgreen'>辅助瞄准范围修正</font><br>
<font color='orange'>ZonesScalingConfig</font><font color='aqua'>(FUASZonesScalingConfig)</font>:<font color='lightgreen'>辅助瞄准范围修正</font>配置项<br>
<font color='orange'>bAutoAimConfig</font><font color='aqua'>(bool)</font>:是否开启<font color='lightgreen'>自动瞄准</font><br>
<font color='orange'>AutoAimConfig</font><font color='aqua'>(FUASAutoAimConfig)</font>:<font color='lightgreen'>自动瞄准</font>配置项<br>
<font color='orange'>CrosshairOffset</font><font color='aqua'>(FVector2D)</font>:默认准心的屏幕位置偏移（一般为0）<br>
</details>

<details>
<summary>Stickiness(灵敏度修正配置项FUASStickinessZoneConfig)</summary>
<font color='orange'>Radius</font><font color='aqua'>(float)</font>:触发修正灵敏度的范围<br>
<font color='orange'>Multiplier</font><font color='aqua'>(float)</font>:灵敏度系数<br>
<font color='orange'>StickinessMultiplierCurvePitch</font><font color='aqua'>(FRuntimeFloatCurve)</font>:俯仰角系数曲线<br>
<font color='orange'>StickinessMultiplierCurveYaw</font><font color='aqua'>(FRuntimeFloatCurve)</font>:偏航角系数曲线<br>
</details>

<details>
<summary>Magnetism(磁性子弹配置项FUASMagnetismZoneConfig)</summary>
<font color='orange'>StartRadius</font><font color='aqua'>(float)</font>:开始触发磁性子弹的范围<br>
<font color='orange'>AimZoneRadius</font><font color='aqua'>(float)</font>:停止磁性子弹修正的范围<br>
</details>

<details>
<summary>Scaling(辅助瞄准范围修正配置项FUASZonesScalingConfig)</summary>
<font color='orange'>ZonesScalingCurve</font><font color='aqua'>(FRuntimeFloatCurve)</font>:触发辅助瞄准范围修正系数曲线<br>
</details>

<details>
<summary>AutoAim(自动瞄准配置项FUASZonesScalingConfig)</summary>
<font color='orange'>Speed</font><font color='aqua'>(float)</font>:瞄准速度<br>
<font color='orange'>AutoAimZoneRadius</font><font color='aqua'>(float)</font>:触发自动瞄准的屏幕范围<br>
<font color='orange'>ActivationDistance</font><font color='aqua'>(float)</font>:触发自动瞄准的目标距离<br>
<font color='orange'>TimeWithoutCameraInputToEnableAutoAim</font><font color='aqua'>(float)</font>:玩家停止相机输入后x秒开始自动瞄准<br>
<font color='orange'>TimeWithMovementInputToEnableAutoAim</font><font color='aqua'>(float)</font>:玩家开始移动后x秒开始自动瞄准<br>
<font color='orange'>bUseOnlyWithInactiveMagnetism</font><font color='aqua'>(bool)</font>:是否和磁性子弹互斥<br>
<font color='orange'>TimeToBlockAfterChangeTarget</font><font color='aqua'>(float)</font>:切换目标后x秒停止自动瞄准<br>
</details>

#### <font color='red'>UASAimAssistTargetComponent</font>
<details>
<summary>目标信息配置</summary>
<font color='orange'>bIsAimAssistActive</font><font color='aqua'>(bool)</font>:是否可被辅助瞄准<br>
<font color='orange'>AimTargetSocketNames</font><font color='aqua'>(TArray<FName>)</font>:目标上的sockets<br>
</details>

## 三、功能说明

### 3.1通用功能

#### <font color='red'>**获取目标列表**</font>
- 刷新频率(UpdateTargetsRate):按这个时间来更新周围目标
- 检测范围(AimAreaExtents)：以（相机位置+相机朝向*长方形长）为中心的长方形。用overlap方式获取所有actor
- 获取actor的全部UUASAimAssistTargetComponent组件
- 获取组件里的全部sockets位置
- 筛选掉与玩家相机阻挡的sockets
- 比较获得距离玩家屏幕中心最近的socket
- 记录最终的<font color='red'>目标</font>以及<font color='red'>socket</font>信息
~~~mermaid
graph LR
	BeginPlay-->SetAimAssistDataAsset
	SetAimAssistDataAsset-->A["SetTimer(UpdateAssist)"]
	A-->B[UpdateTargetsHandleTargets]
~~~
<details>
<summary><font color='lightblue'>获取TargetComponent代码</font></summary>

```c++
void UMyAimAssistComponenet::UpdateTargets()
{
	SCOPE_CYCLE_COUNTER(STAT_UpdateTargets);

	TArray<FOverlapResult> overlapResults;
	FCollisionQueryParams collisionParams;
	collisionParams.AddIgnoredActor(PlayerController->GetPawn());

	GetWorld()->OverlapMultiByProfile(
		overlapResults,
		GetOverlapLocation(),
		GetOverlapRotation().Quaternion(),
		TargetsDetectionProfileName,
		FCollisionShape::MakeBox(GetOverlapExtents()),
		collisionParams);

	TArray<TWeakObjectPtr<AActor>> targetsToRemove;
	TargetComponents.GenerateKeyArray(targetsToRemove);

	for (const auto& result : overlapResults)
	{
		if (result.GetActor() == nullptr)
		{
			continue;
		}

		if (TargetComponents.Contains(result.GetActor()))
		{
			targetsToRemove.Remove(result.GetActor());
			continue;
		}

		TInlineComponentArray<UUASAimAssistTargetComponent*> targetComponents;
		result.GetActor()->GetComponents(targetComponents);

		if (targetComponents.Num() != 0)
		{
			auto& val = TargetComponents.Add(TWeakObjectPtr<AActor>(result.GetActor()));

			result.GetActor()->OnDestroyed.AddDynamic(this, &UMyAimAssistComponenet::OnTargetDestroyed);

			for (const auto component : targetComponents)
			{
				val.Add(component);
			}
		}
	}

	for (const auto actor : targetsToRemove)
	{
		actor->OnDestroyed.RemoveAll(this);
		TargetComponents.Remove(actor);
	}
}
```
</details>

<details>
<summary><font color='lightblue'>筛选所有socket代码</font></summary>

```c++
void UMyAimAssistComponenet::HandleTargets()
{
	SCOPE_CYCLE_COUNTER(STAT_HandleTargets);

	LastTargetData.Empty(TargetComponents.Num());

	const FVector viewLocation = GetCameraLocation();

	for (const auto& target : TargetComponents)
	{
		if (!target.Key.IsValid())
		{
			continue;
		}

		for (const auto component : target.Value)
		{
			if (!component.IsValid() || !component->IsTargetActive())
			{
				continue;
			}

			FHitResult hitResult;

			for (const auto& socketData : component->GetAimTargetSocketLocations())
			{
				UKismetSystemLibrary::LineTraceSingleByProfile(this,
					viewLocation,
					socketData.Location,
					ObstacleCheckProfileName,
					false,
					{ GetOwner(), target.Key.Get() },
					bDebugTargetTraces ? EDrawDebugTrace::ForDuration : EDrawDebugTrace::None,
					hitResult,
					true);

				if (hitResult.bBlockingHit)
				{
					continue;
				}

				LastTargetData.Add({ component, socketData.SocketName });
			}
		}
	}

	if (LastTargetData.Num() != 0)
	{
		CurrentTargetData = *Algo::MinElementBy(LastTargetData, [&](const FUASAimAssistTargetData& Data) {
			auto location = Data.GetLocationOnTheScreen(PlayerController.Get());
		return (location - GetScreenCenter()).Size();
			});
	}
	else
	{
		CurrentTargetData = {};
	}
}
```
</details>

### 3.2灵敏度修正

#### <font color='red'>触发条件</font>
$$distant = |目标socket屏幕中位置-准星位置| $$
$$ distant\leq Radius(触发修正灵敏度的范围)$$
#### <font color='red'>修正系数</font>
整体系数：<font color='red'>Multiplier</font>

俯仰/偏航角修正系数：
$$y_{pitch}(x)=Curve_{pitch}(x)*Multiplier$$
$$y_{yaw}(x)=Curve_{yaw}(x)*Multiplier$$
$$x=1-distant/Radius$$
#### <font color='red'>总结</font>
越靠近中心灵敏度越低

### 3.3磁性子弹

#### <font color='red'>触发条件</font>
$$ distant\leq StartRadius(触发磁性子弹的范围)$$
#### <font color='red'>修正方法</font>
中心范围：<font color='red'>magnetismRadius</font>

俯仰/偏航角修正系数：
$$lenght = |\vec{targetLocation} - \vec{targetCrosshairPosition}|$$
当`length>magnetismRadius`时：
$$\vec{targetCrosshairPosition} = \vec{targetCrosshairPosition} - \hat{(\vec{targetCrosshairPosition} - \vec{targetLocation})} * magnetismRadius$$
$$\vec{targetCrosshairPosition}:准星当前位置$$
$$\vec{targetLocation}:目标socket在屏幕上的位置$$
$$\vec{targetCrosshairPosition}:准星当前位置$$

当目标消失时，targetCrosshairPosition直接回归到默认中心位置
#### <font color='red'>总结</font>
靠近目标后准信会被自动拉向目标socket上。

### 3.3辅助瞄准范围修正

#### <font color='red'>触发条件</font>
$$ 有目标socket存在$$
#### <font color='red'>修正方法</font>
所有的范围类配置都有个修正：<font color='red'>ZonesScaleMultiplier</font>

具体系数：
$$distanceToTarget = |目标socket世界坐标 - 玩家世界坐标|$$
$$ZonesScaleMultiplier = ZonesScalingCurve(distanceToTarget)(读曲线上该点的值)$$

当目标消失时，ZonesScaleMultiplier直接回归为1.0
#### <font color='red'>总结</font>
各种范围会随着玩家远离目标而减小。

### 3.4自动瞄准

#### <font color='red'>触发条件</font>
$$
distant\leq AutoAimZoneRadius(触发自动瞄准的屏幕范围)\\
\&\ |目标socket世界坐标 - 玩家世界坐标|\leq ActivationDistance\\
\& 连续没有Controller输入的时间\geq TimeWithoutCameraInputToEnableAutoAim\\
\& 连续有移动输入的时间\geq TimeWithMovementInputToEnableAutoAim
$$
#### <font color='red'>修正方法</font>
```c++
targetRotation = UKismetMathLibrary::RInterpTo(PlayerController->GetControlRotation(), targetRotation, DeltaTime, AimAssistDataAsset->AutoAimConfig.Speed)
```
$$
targetRotation:从玩家相机到目标socket的旋转\\
Speed：旋转速度
$$

#### <font color='red'>总结</font>
- 玩家移动时如果不主动瞄准会自动跟枪