# MySQL Connection Pool
### Cannot enqueue Handshake after invoking quit

Instead of using a single mysql connection per lambda invocation if you are expecting to call that function
multiple times consequtively it's better to create a connection pool *outside* of the handler.

In serverless, variables outside of the handler *may* (not always) persist between invocations.
Therefore creating a single mysql connection _mysql.createConnection()_ outside of the handler and then closing 
that connection inside of the handler could lead to the error "Cannot enqueue Hnadshake after invoking quit"

``` javascript
const pool = mysql.createPool({
	host: process.env.DBHOST,
	user: process.env.DBUSER,
	password: process.env.DBPASSWORD,
	database: process.env.DBNAME
})

module.exports.processSpreadSheet = (event, context, callback) => {
	pool.getConnection((err, connection) => {
		if(err) {
			// ...handle your err
		} else {
			connection.query('SELECT * FROM the_stars', (err, res) => {
				connection.release()
			})
		}
	})
}
```