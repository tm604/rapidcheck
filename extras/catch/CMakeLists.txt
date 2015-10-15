target_include_directories(rapidcheck INTERFACE include)
install(
	DIRECTORY include/
	DESTINATION "include"
)
