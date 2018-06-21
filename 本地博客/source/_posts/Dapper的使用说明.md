
---
title: Dapper的使用说明
date: 2018-06-19 22:37:21
tags:  #dapper,C#
keywords:  #dapper,C#
---
## 1.表结构
  ![Image text](https://raw.githubusercontent.com/hongmaju/blog/master/本地博客/source/_posts/images/Dapper的使用说明1.png)
    ![Image text](https://raw.githubusercontent.com/hongmaju/blog/master/本地博客/source/_posts/images/Dapper的使用说明2.png)
## 2、程序对应的实体类
  ![Image text](https://raw.githubusercontent.com/hongmaju/blog/master/本地博客/source/_posts/images/Dapper的使用说明3.png)
    ![Image text](https://raw.githubusercontent.com/hongmaju/blog/master/本地博客/source/_posts/images/Dapper的使用说明4.png)
## 3、基本操作
* 3.1  插入
~~~
        public int Insert(Person person, string _ConnString)
        {
            using (IDbConnection connection = new SqlConnection(_ConnString))
            {
                return connection.Execute("insert into Person(Name,Remark) values(@Name,@Remark)", person);

            }

        }
~~~
* 3.2  删除
 ~~~
         public int Delete(Person person, string connectionString)
         {
             using (IDbConnection connection = new SqlConnection(connectionString))
             {
                 return connection.Execute("delete from Person where id=@ID", person);
             }
         }
 ~~~

* 3.3 修改
~~~
        public int Update(Person person, string connectionString)
        {
            using (IDbConnection connection = new SqlConnection(connectionString))
            {
                return connection.Execute("update Person set name=@Name where id=@ID", person);
            }
        }
~~~

* 3.4 查询
~~~
        /// <summary>
        /// 批量修改
        /// </summary>
        /// <param name="persons"></param>
        /// <param name="connectionString"></param>
        /// <returns></returns>
        public int Update(List<Person> persons, string connectionString)
        {
            using (IDbConnection connection = new SqlConnection(connectionString))
            {
                return connection.Execute("update Person set name=@name where id=@ID", persons);
            }
        }


        /// <summary>
        /// 无参查询所有数据
        /// </summary>
        /// <returns></returns>
        public List<Person> Query(string connectionString)
        {
            using (IDbConnection connection = new SqlConnection(connectionString))
            {
                return connection.Query<Person>("select * from Person").ToList();
            }
        }
~~~

## 4、其他操作
* 4.1 批量操作
~~~
        public int Insert(List<Person> persons, string connectionString)
        {
            using (IDbConnection connection = new SqlConnection(connectionString))
            {
                var transaction = connection.BeginTransaction();
                var rowsAffectd = 0;
                try
                {

                    rowsAffectd = connection.Execute("insert into Person(Name,Remark) values(@Name,@Remark)", persons, transaction);
                    transaction.Commit();
                    return rowsAffectd;
                }
                catch (Exception)
                {
                    transaction.Rollback();
                    throw;
                }
            }
        }
~~~

* 4.2 in操作
~~~
        public List<Person> QueryIn(string connectionString)
        {
            using (IDbConnection connection = new SqlConnection(connectionString))
            {
                var sql = "select * from Person where id in @ids";
                //参数类型是Array的时候，dappper会自动将其转化
                return connection.Query<Person>(sql, new { ids = new int[2] { 1, 2 }, }).ToList();
            }
        }

         public List<Person> QueryIn(int[] ids, string connectionString)
                {
                    using (IDbConnection connection = new SqlConnection(connectionString))
                    {
                        var sql = "select * from Person where id in @ids";
                        //参数类型是Array的时候，dappper会自动将其转化
                        return connection.Query<Person>(sql, new { ids }).ToList();
                    }
                }
~~~

* 4.3 返回多张表的结果
~~~
public IEnumerable<Person> QueryMultiple(string connectionString, ref IEnumerable<Book> bookList)
        {
            using (IDbConnection connection = new SqlConnection(connectionString))
            {
                var sql = "select * from Person; select * from Book";//取表的顺序要一致
                var multiReader = connection.QueryMultiple(sql);
                var personList = multiReader.Read<Person>();
                bookList = multiReader.Read<Book>();

                multiReader.Dispose();
                return personList;

            }
        }

~~~
* 4.4  事务
~~~
 public void TransactionExecuteCommand(string connectionString)
        {
            using (IDbConnection connection = new SqlConnection(connectionString))
            {
                connection.Open();
                IDbTransaction tran = connection.BeginTransaction();

                try
                {
                    connection.Execute("delete from Person where Id=9", transaction: tran);
                    tran.Commit();
                }
                catch
                {

                    tran.Rollback();
                    throw new Exception();
                }

            }
        }
~~~

* 4.5 分页查询
引用Dapper.extend.dll，使用其中的SqlBuilder方法，方便生成sql语句
~~~
        /// <summary>
        /// 分页查询
        /// </summary>
        /// <param name="pageIndex">页面序号（以0开始）</param>
        /// <param name="pageSize">页面大小</param>
        /// <param name="asc">行数的排序</param>
        /// <param name="desc">行数的排序</param>
        /// <param name="connectionString">连接字符串</param>
        /// <param name="whereID">查询条件id，其他的条件往后加</param>
        /// <returns></returns>
        public Tuple<IEnumerable<Person>, int> Find(int pageIndex, int pageSize, string[] asc, string[] desc, string connectionString, string whereID)
        {


            using (IDbConnection connection = new SqlConnection(connectionString))
            {
                const string countQuery = @"SELECT COUNT(1)
                              FROM   [Person]    /**where**/";

                const string selectQuery = @" SELECT *
                      FROM  ( SELECT  ROW_NUMBER() OVER ( /**orderby**/ ) AS RowNum,  p.*
                           FROM   [Person] p    /**where**/)
                        AS RowConstrainedResult
                      WHERE  RowNum >= (@PageIndex * @PageSize + 1 )
                        AND RowNum <= (@PageIndex + 1) * @PageSize
                      ORDER BY RowNum ";

                SqlBuilder builder = new SqlBuilder();

                var count = builder.AddTemplate(countQuery);
                var selector = builder.AddTemplate(selectQuery, new { PageIndex = pageIndex, PageSize = pageSize });

                foreach (var a in asc)
                {
                    if (!string.IsNullOrWhiteSpace(a))
                        builder.OrderBy(a);
                }

                foreach (var d in desc)
                {
                    if (!string.IsNullOrWhiteSpace(d))
                        builder.OrderBy(d + " desc");
                }

                if (!string.IsNullOrEmpty(whereID))
                    builder.Where("id>= @Id", new { Id = whereID });


                var totalCount = connection.Query<int>(count.RawSql, count.Parameters).Single();
                var rows = connection.Query<Person>(selector.RawSql, selector.Parameters);

                return new Tuple<IEnumerable<Person>, int>(rows, totalCount);
            }
        }
~~~

* 4.6 联合查询
  ![Image text](https://raw.githubusercontent.com/hongmaju/blog/master/本地博客/source/_posts/images/Dapper的使用说明5.png)
    ![Image text](https://raw.githubusercontent.com/hongmaju/blog/master/本地博客/source/_posts/images/Dapper的使用说明6.png)
      ![Image text](https://raw.githubusercontent.com/hongmaju/blog/master/本地博客/source/_posts/images/Dapper的使用说明7.png)
~~~
/// <summary>
        /// join语句,联合查询
        /// </summary>
        /// <param name="book"></param>
        /// <param name="connectionString"></param>
        /// <returns></returns>
        public List<BookWithPerson> QueryJoin(Book book, string connectionString)
        {
            using (IDbConnection connection = new SqlConnection(connectionString))
            {
                var sql = @"select b.id,b.bookName,p.id,p.name,p.remark
                        from Person as p
                        join Book as b
                        on p.id = b.personId
                        where b.id = @id;";
                var result = connection.Query<BookWithPerson, Person, BookWithPerson>(sql,
                    (bookWithPerson, person) =>
                    {
                        bookWithPerson.Pers = person;
                        return bookWithPerson;
                    },
                    book, splitOn: "id");
                //splitOn: "bookName");
                return (List<BookWithPerson>)result;
            }
        }

          //其中，Query的三个泛型参数分别是委托回调类型1，委托回调类型2，返回类型。
                //形参的三个参数分别是sql语句，map委托，对象参数。
                //所以整句的意思是先根据sql语句查询；同时把查询的person信息赋值给bookWithPerson.Pers，
                //并且返回bookWithPerson；book是对象参数，提供参数绑定的值。最终整个方法返回BookWithPerson
~~~
* 4.7 字段映射
  ![Image text](https://raw.githubusercontent.com/hongmaju/blog/master/本地博客/source/_posts/images/Dapper的使用说明8.png)
    ![Image text](https://raw.githubusercontent.com/hongmaju/blog/master/本地博客/source/_posts/images/Dapper的使用说明9.png)
      ![Image text](https://raw.githubusercontent.com/hongmaju/blog/master/本地博客/source/_posts/images/Dapper的使用说明10.png)
~~~
 public List<BookType> QueryBookType(string connectionString)
        {
            using (IDbConnection connection = new SqlConnection(connectionString))
            {

                SqlMapper.SetTypeMap(typeof(BookType), new ColumnAttributeTypeMapper<BookType>());
                return connection.Query<BookType>("select * from BookType").ToList();
            }
        }

        public List<BookType> QueryBookTypeAS(string connectionString)
        {
            using (var conn = new SqlConnection(connectionString))
            {
                List<BookType> type = conn.Query<BookType>("select id asBookID,bookType as Type from BookType").ToList();
                return type;
            }

        }

        /// <summary>
        /// 使用linq
        /// </summary>
        /// <param name="connectionString"></param>
        /// <returns></returns>
        public List<BookType> QueryBookTypeLinq(string connectionString)
        {
            using (var conn = new SqlConnection(connectionString))
            {
                List<BookType> type = conn.Query<dynamic>("select * from BookType")
                                          .Select(item => new BookType()
                                          {
                                              BookID = item.id,
                                              Type = item.bookType
                                          })
                                          .ToList();

                return type;
            }
        }
~~~
