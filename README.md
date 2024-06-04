# ECS
# [특징]
 - ECS는 데이터와 로직이 완벽하게 분리되어있음
 - ECS의 기능을 사용하는 객체는 SubScene에서만 분리되어 사용 가능하다.

   
# [Detail]
1. IComponentData
   - ECS는 IComponentData라는 Interface를 상속받은 Struct를 컴포넌트별 데이터로 정의
     ![image](https://github.com/xowjd913/resume/assets/25768804/f01dc8a8-a026-4c0b-8bbe-e23d5ed2db67)
2. Authoring
   - Authoring 클래스 안에는 변환 처리를 위한 Baker 클래스가 정의됨
   - 해당 내용은 Unity Editor 상에서 설정된 갑을 Dancer 컴포넌트로 변환하는 처리만함.
     ![image](https://github.com/xowjd913/resume/assets/25768804/183a8cf0-0494-4aa2-b66f-5364d45dc791)
3. System
   - ISystem Interface를 상속받은 Struct로 구현됨
   - 해당 Struct는 partial로 정의됨
     - ECS는 C#의 Source Generator로 다양한 보조적 코드를 자동생성하는데 자동생성된 코드를 같이 사용하기위해 필요
   - 현재 경과 시간을 얻기 위해서는 SystemAPI의 Time에서 가져올 수 있음
   - SystemAPI의 Query를 통해 Entity를 가져올 수 있음. 기존의 GetComponent와 비슷..
     - Entity가져올 때 RefRO / RefRW가 있음.
       - RO = ReadOnly / RW = ReadWrite
  ![image](https://github.com/xowjd913/resume/assets/25768804/bd405e6b-4b27-42e9-94e3-b55c10562ba1)

4. SubScene
   - Hierachy에서 우클릭 후 ContextMenu를 통해 SubScene을 생성 가능
     ![image](https://github.com/xowjd913/resume/assets/25768804/e5f6ee74-6555-4348-931c-2a590e8afa61)
   - 일반적인 OOP 방식 개발할때 객체를 Scene에 배치한것처럼 동일하게 SubScene에 배치하면됨
     ![image](https://github.com/xowjd913/resume/assets/25768804/3711899f-5755-48cf-b345-ab0be94a5239)

5. 샘플 [ 회전 큐브 ]
   - 위 내용대로 차근차근 진행해보면, 우선 기능별로 Assets폴더에 폴더구성을 하는게 좋을듯하여 Scripts/ECS/Rotation를 만들어보자.
     ![image](https://github.com/xowjd913/resume/assets/25768804/1c818dc7-93fa-4c57-b915-262697a018ec)
   - 위 내용대로 ComponentData를 만들어 객체를 회전할때 필요한 변수를 선언하자 
  ```
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

using Unity.Entities;
using Unity.Mathematics;

public struct RotationSpeedComponentData : IComponentData
{
    public float RadiansPerSecond;
}
  ```
   - 이제 Authoring 코드를 작성해 Baker를 만들어보자
  ```
using Unity.Entities;
using Unity.Mathematics;

public class RotationAuthoring : MonoBehaviour
{
    public float degreePerSecond = 360f;

    class Baker : Baker<RotationAuthoring>
    {
        public override void Bake(RotationAuthoring authoring)
        {
            var entity = GetEntity(TransformUsageFlags.Dynamic | TransformUsageFlags.NonUniformScale);
            AddComponent(entity: entity, new RotationComponentData() 
            {
                RadiansPerSecond = math.radians(authoring.degreePerSecond)
            });
        }
    }
}
  ```
  - 이제 System을 만들어서 로직을 동작시켜보자.
  ```
using Unity.Entities;
using Unity.Transforms;

public partial struct RotationSystem : ISystem
{
    public void OnUpdate(ref SystemState state) 
    {
        float deltaTime = SystemAPI.Time.DeltaTime;

        // Get Query
        //var localTransform = SystemAPI.Query<RefRW<LocalTransform>>();
        //var rotationSpeedComponent = SystemAPI.Query<RefRO<RotationComponentData>>();

        // Get Query
        foreach (var (transform, speed) in SystemAPI.Query<RefRW<LocalTransform>, RefRO<RotationComponentData>>())
        {
            transform.ValueRW = transform.ValueRO.RotateY(speed.ValueRO.RadiansPerSecond * deltaTime);
        }
    }
}
  ```
 - 그럼 이제 따로 코딩은 끝. 돌리고 싶은 객체에 Authoring을 Add해주자.
   ![image](https://github.com/xowjd913/study/assets/25768804/2f78bc21-9218-4191-b444-f3b208df97fc)
 - 실행 해보면 Scene View에서는 아무런 움직임이 없으나 Game View에서는 자동으로 Rotation이 적용되는걸 확인할 수 있다.
   ![image](https://github.com/xowjd913/study/assets/25768804/cd3b1cb0-3418-4001-9254-687195917695)


   [개인적인 의견]
    - 메인 씬에서 기존의 로직대로 돌리고, ECS로 동작해야될 부분은 SubScene에서 동작 시키면 리소스를 벌 수는 있을것같으나, 코드가 난잡해질듯?
      
   


