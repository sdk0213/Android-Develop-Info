Observable 중간상태값 확인
===
* 함수형 프로그래밍 패러다임에서는 "로그나 화면 출력하는 등 부수 효과를 발생시킨다", 때문에 최소화하려고 하는 경향이 있다.
* **하지만 적절한 로그느 유지 보수성을 확보하기 위해 필요하다.**
* 실행시 찜찜한 부분은 doOnNext(), doOnComplete(), doOnError() 함수를 추가한다. -> 7장 참고
