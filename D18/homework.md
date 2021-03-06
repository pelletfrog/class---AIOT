# 作業

## 作業1

* 使用 Flask 撰寫一個會員系統，將會員資料存入 MongoDB，並提供 4 種 API 操作 MongoDB 中的資料，分別是新增、修改、刪除、查詢
* Ans:

```python
	from flask import Flask, request, jsonify
	from flask_pymongo import PyMongo
	from bson.objectid import ObjectId

	app = Flask(__name__)
	app.config['MONGO_URI'] = "mongodb://127.0.0.1:8088/test" # test 為資料庫名稱
	mongo = PyMongo(app)

	# 查詢
	@app.route('/members')
	@app.route('/member/<id>', methods = ['GET'])
	def get_member(id=None):
	    # 若沒有傳入 id，即取得所有會員資料
	    if id is None:
	        members = mongo.db.member.find({})
	        result = [] # 宣告一個陣列，回傳結果時需要用到
	        
	        for member in members:
	            member['_id'] = str(member['_id']) # mongo 回傳的 id 為 ObjectId 型別，把 id 轉成一般的字串
	            result.append(member)
	        return jsonify(result) # 把我們的結果轉換為 json 並回傳
	    else:
	        result = mongo.db.member.find_one({'_id': ObjectId(id)})
	        if result is not None:
	            result['_id'] = str(result['_id']) # mongo 回傳的 id 為 ObjectId 型別，把 id 轉成一般的字串
	        
	        return jsonify(result) # 把我們的結果轉換為 json 並回傳


	# 新增
	@app.route('/members', methods = ['POST'])
	def add_member():
	    # 把 form 裡面的資料拿出來
	    name = request.form.get('name')
	    age = request.form.get('age')
	    phone = request.form.get('phone')
	    email = request.form.get('email')
	    
	    # 執行 insert_one 新增一筆資料
	    result = mongo.db.member.insert_one({'name': name, 'age': age, 'phone': phone, 'email': email})
	    
	    # 回傳新增資料的 id
	    return str(result.inserted_id)


	# 修改
	@app.route('/members/<id>', methods = ['PUT'])
	def update_member(id):
	    result = 0
	    
	    # 把 form 裡面的資料拿出來
	    name = request.form.get('name')
	    age = request.form.get('age')
	    phone = request.form.get('phone')
	    email = request.form.get('email')
	    
	    # 把新的資料用 dict 型別包裝
	    new_value = { "$set": { "name": name, "age": age, "phone": phone, "email": email } }
	    # 執行 update_one 進行修改
	    upd_result = mongo.db.member.update_one({"_id": ObjectId(id)}, new_value)
	    
	    if upd_result is not None:
	        result = upd_result.modified_count
	    
	    # 回傳更新的資料筆數
	    return "Update %s data" % str(result)


	# 刪除
	@app.route('/members/<id>', methods = ['DELETE'])
	def remove_member(id):
	    result = 0
	    
	    # 先確定是否有該會員
	    member = mongo.db.member.find_one({'_id': ObjectId(id)})
	    
	    # 若會員存在，執行 delete_one 刪除該會員
	    if member is not None:
	        del_result = mongo.db.member.delete_one(member)
	        result = del_result.deleted_count
	        
	    # 回傳刪除的資料筆數
	    return "Delete %s data" % str(result)



	if __name__ == '__main__':
		app.run()
```