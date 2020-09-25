## 이클립스에서 java + gradle로 개발 중 오류 발생

### 오류내용
```
Could not run phased build action using Gradle distribution 'https://services.gradle.org/distributions/gradle-6.0-bin.zip'
```

### 오류해결
1. 오류가 발생한 프로젝트의 Navigator View에서 .project 파일 열기
2. 'org.eclipse.buildship.core.gradleprojectbuilder'와 'org.eclipse.buildship.core.gradleprojectnature'를 값으로 가진 태그 삭제
3 .gradle/ 디렉토리 삭제
4. 전체 프로젝트 클린 및 새로고침
5. Gradle Tasks View에서 해당 프로젝트 gradle project로 import 

### 결론
정확한 원인은 모르겠지만 위 과정을 수행했더니 해당 오류가 사라졌음.
