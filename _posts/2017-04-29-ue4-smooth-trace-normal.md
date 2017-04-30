---
layout: post
title:  "UE4 Smoothed Normal from Line Trace in Packaged Builds"
date:   2017-04-29 10:10:00 +0100
categories: ['gamedev', 'programming', 'ue4']
---

I needed a way to get a smoothed surface normal from a line trace in Unreal Engine 4. So of course the first thing I did was research the topic. I found a lot of good resources that I linked at the bottom of this post. However none of the forum posts mentioned the solution to accessing mesh data in a packaged build. This might've been because I was searching the wrong keywords. I included my code to calculate the interpolated surface normal for both Static Meshes and Spline Meshes in Unreal Engine 4. 

<!--more-->

Most of the resources I found were talking about getting UV coordinates for a Line Trace hit. And many didn't consider Spline Meshes at all which is my primary usecase. And one important thing I didn't find mentioned anywhere is that you need to enable Allow CPUAccess on static meshes that you want to access during runtime in a packaged build. I found that one while digging through the source code, although it's a pretty obvious thing in hindsight. 

{% include inline-image.html src="2017-04-smoothnormals/allowcpuaccess.png" description="To be able to access mesh data in a packaged build bAllowCPUAccess must to be set to true." %}

The following code is part of my static c++ function library. It accepts a FHitResult reference and calculates an interpolated normal based on the triangle that was hit. This works for both Static Mesh as well as Spline Mesh Components. However this code is by no means perfect and I am sure there is a lot that could be improved.

{% highlight cpp %}
bool GetSmoothNormalFromTrace(const FHitResult& HitResult, FVector& Normal, bool DebugDraw)
{
    // We don't care if this isn't a blocking hit
    if(!HitResult.bBlockingHit)
        return false;

    // See if a static mesh and/or a spline mesh component were hit
    UStaticMeshComponent* StaticMeshComp = Cast<UStaticMeshComponent>(HitResult.GetComponent());
    USplineMeshComponent* SplineMeshComp = Cast<USplineMeshComponent>(HitResult.GetComponent());

    // If we hit a spline mesh component we should also have hit a static mesh component
    if(!StaticMeshComp) return false;

    // Retrieve static mesh. If we hit a spline mesh component we also hit a static mesh component at the same time.
    // So we can just use it to get to the actual static mesh.
    UStaticMesh* StaticMesh = StaticMeshComp->GetStaticMesh();

    // Return if the static mesh isn't set (shouldn't happen)
    if(!StaticMesh) 
        return false;

    // In cooked builds we need to have this flag set or we'll crash when trying to access mesh data
    if(!StaticMesh->bAllowCPUAccess)
        return false;

    // Return if RenderData is invalid
    if(!StaticMesh->RenderData)
        return false;

    // No valid mesh data on lod 0 (shouldn't happen)
    if(!StaticMesh->RenderData->LODResources.IsValidIndex(0))
        return false;
 
    int FaceIndex = HitResult.FaceIndex;
    FTransform ComponentTransform = StaticMeshComp->GetComponentTransform();
    FStaticMeshVertexBuffer* VertexBuffer = &StaticMesh->RenderData->LODResources[0].VertexBuffer;
    FPositionVertexBuffer* PositionVertexBuffer = &StaticMesh->RenderData->LODResources[0].PositionVertexBuffer;
    FIndexArrayView IndexBuffer = StaticMesh->RenderData->LODResources[0].IndexBuffer.GetArrayView();

    // Storage for the actual triangle verteces
    FVector VertexPositions[3];
    FVector VertexNormals[3];

    for(int i = 0; i < 3; i++) {
        // Get vertex index
        uint32 index = IndexBuffer[FaceIndex * 3 + i];
        // Get vertex position and normal
        VertexPositions[i] = PositionVertexBuffer->VertexPosition(index);
        VertexNormals[i] = VertexBuffer->VertexTangentZ(index);
        
        // Transform position and normal into spline mesh space
        if(SplineMeshComp) {
            // Get transform along spline component
            const FTransform SliceTransform = SplineMeshComp->CalcSliceTransform(USplineMeshComponent::GetAxisValue(VertexPositions[i], SplineMeshComp->ForwardAxis));
            // Remove spline forward axis from vertex position, it will be added back by transforming the position into spline mesh space
            USplineMeshComponent::GetAxisValue(VertexPositions[i], SplineMeshComp->ForwardAxis) = 0;
            // Transform position and normal into spline mesh space
            VertexPositions[i] = SliceTransform.TransformPosition(VertexPositions[i]);
            VertexNormals[i] = SliceTransform.TransformVector(VertexNormals[i]);
        }

        // Transform position and normal into world space
        VertexPositions[i] = ComponentTransform.TransformPosition(VertexPositions[i]);
        VertexNormals[i] = ComponentTransform.TransformVector(VertexNormals[i]);
    }
    
    // Determine the barycentric coordinates
    FVector U = VertexPositions[1] - VertexPositions[0];
    FVector V = VertexPositions[2] - VertexPositions[0];
    FVector W = HitResult.ImpactPoint - VertexPositions[0];

    FVector vCrossW = FVector::CrossProduct(V, W);
    FVector vCrossU = FVector::CrossProduct(V, U);

    if(FVector::DotProduct(vCrossW, vCrossU) < 0.0f) {
        return false;
    }

    FVector uCrossW = FVector::CrossProduct(U, W);
    FVector uCrossV = FVector::CrossProduct(U, V);

    if(FVector::DotProduct(uCrossW, uCrossV) < 0.0f) {
        return false;
    }

    float Denom = uCrossV.Size();
    float b1 = vCrossW.Size() / Denom;
    float b2 = uCrossW.Size() / Denom;
    float b0 = 1.0f - b1 - b2;

    // Determine the hit normal
    Normal.X = b0 * VertexNormals[0].X + b1 * VertexNormals[1].X + b2 * VertexNormals[2].X;
    Normal.Y = b0 * VertexNormals[0].Y + b1 * VertexNormals[1].Y + b2 * VertexNormals[2].Y;
    Normal.Z = b0 * VertexNormals[0].Z + b1 * VertexNormals[1].Z + b2 * VertexNormals[2].Z;
    
    // Just to be safe here
    Normal.Normalize();

    // Debug draw
    #if !UE_BUILD_SHIPPING
    if(DebugDraw) {
    	// Quick and dirty debug draw
        FColor DebugColor = FColor::Red;
        float DebugDrawDuration = 30.0f;

        for(int i = 0; i < 3; i++) {
            // draw triangle points
            ::DrawDebugPoint(StaticMeshComp->GetWorld(), VertexPositions[i], 16, DebugColor, false, DebugDrawDuration);
            // draw triangle edges
            ::DrawDebugLine(StaticMeshComp->GetWorld(), VertexPositions[i], VertexPositions[(i+1)%3], DebugColor, false, DebugDrawDuration);
            // triangle normals
            ::DrawDebugLine(StaticMeshComp->GetWorld(), VertexPositions[i], VertexPositions[i] + VertexNormals[i]  * 200.0f, DebugColor, false, DebugDrawDuration);
            }

        // draw actual impact normal
        ::DrawDebugPoint(StaticMeshComp->GetWorld(), HitResult.ImpactPoint, 16, DebugColor, false, DebugDrawDuration);
        ::DrawDebugLine(StaticMeshComp->GetWorld(), HitResult.ImpactPoint, HitResult.ImpactPoint + HitResult.ImpactNormal * 200.0f, DebugColor, false, DebugDrawDuration);
        ::DrawDebugLine(StaticMeshComp->GetWorld(), HitResult.ImpactPoint, HitResult.ImpactPoint + Normal * 200.0f, DebugColor, false, DebugDrawDuration);
    }
    #endif //!UE_BUILD_SHIPPING
    
    return true;
}
{% endhighlight %}


Resources used:

* [UE4 Forum - Question about traces and geometry.](https://forums.unrealengine.com/showthread.php?62805-Question-about-traces-and-geometry)
* [UE4 Forum - Accessing Vertex Positions of static mesh](https://forums.unrealengine.com/showthread.php?8856-Accessing-Vertex-Positions-of-static-mesh)
* [UE4 Forum - How to get vertex positions of spline mesh component in world space?](https://forums.unrealengine.com/showthread.php?138508-How-to-get-vertex-positions-of-spline-mesh-component-in-world-space)
* [UE4 Forum - Accessing Vertex Positions of static mesh](https://forums.unrealengine.com/showthread.php?8856-Accessing-Vertex-Positions-of-static-mesh)
* [UE4 AnswerHub - How can I get polygon normals of a static mesh?](https://answers.unrealengine.com/questions/459327/how-can-i-get-polygon-normals-of-a-static-mesh.html)
* [UE4 AnswerHub - How to get triangles of a Static Mesh?](https://answers.unrealengine.com/questions/340304/how-to-get-triangles-of-a-static-mesh.html)
* [UE4 AnswerHub - How to use Procedural Mesh Component in Blueprint?h](https://answers.unrealengine.com/questions/295318/how-to-use-procedural-mesh-component-in-blueprint.html)
* [UE4 Source Code - Transforming Positions Into SplineMesh Space](https://github.com/EpicGames/UnrealEngine/blob/76085d1106078d8988e4404391428252ba1eb9a7/Engine/Source/Programs/UnrealLightmass/Private/Lighting/StaticMesh.cpp#L151)
* [UE4 Source Code - GetSectionFromStaticMesh](https://github.com/EpicGames/UnrealEngine/blob/76085d1106078d8988e4404391428252ba1eb9a7/Engine/Plugins/Runtime/ProceduralMeshComponent/Source/ProceduralMeshComponent/Private/KismetProceduralMeshLibrary.cpp#L372)
* [UE4 Wiki - Accessing mesh triangles and vertex positions in build](https://wiki.unrealengine.com/Accessing_mesh_triangles_and_vertex_positions_in_build) (Although this one is a bit older)



