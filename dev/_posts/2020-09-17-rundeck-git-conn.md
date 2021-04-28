# rundeck - git 연동
- rundeck plugins에서 git import, export 설치
- scm 연동
  - rundeck docker 의 ssh key 생성해서 repo 에 등록
  - rundeck 의 user 정보를 모두 입력해야 export(commit)가 가능함.(first name, last name)
  - 단, import 를 해야할 때 rundeck 에서 읽을 수 있는 xml 로 생성해야 함.(template 는 있음)
  - uuid 가 필요. 그냥 generate 해서 사용 중 