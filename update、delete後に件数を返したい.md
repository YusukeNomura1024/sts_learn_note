```
	@Override
	public int update(Task task) {
		return jdbcTemplate.update("UPDATE task SET type_id = ?, title = ?, detail = ?,deadline = ? WHERE id = ?",
				task.getTypeId(), task.getTitle(), task.getDetail(), task.getDeadline(), task.getId() );
	}

	@Override
	public int deleteById(int id) {
		return jdbcTemplate.update("DELETE FROM task WHERE id = ?", id);
	}
```
* 上記のようにreturn を記述すれば、戻り値をintにして件数が返るようにできる。
* 0件でも、例外が発生せずに０という数字が返ってくるだけなので、サービスなどで、件数が０だった場合に、例外を発生させるなどの処理が必要になってくる。
