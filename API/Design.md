cda-api design documentation

High Level Overview:


General Pipeline: 
User utilizes cdapython functions to construct a query
cdapython generates an appropriate api call using the constructs defined in cda_client
The api call is sent to CDA API which is built utilizing FastAPI to manage the various endpoints
CDA API processes this api call and generates a SQL query via SQLAlchemy
This query is sent to the CDA Database
The CDA Database processes this query and returns the resulting data to CDA API
CDA API structures the results and returns the data to cdapython



How we utilize FastAPI and SQLAlchemy:
FastAPI
General Concepts
Enables us to quickly stand up an API with easy-to-define endpoints
Leverages Uvicorn to generate a web server that it sits on
Uses Pydantic for input and output data validation
Includes easy-to-use integrations with SQLAlchemy
Automatically generates OpenAPI documentation
Automatically hosts Swagger (http(s)://API_URL/docs) and ReDoc (http(s)://API_URL/redoc) endpoints
Associated Files
cda_api/main.py
Key Concepts:
This is base file that contains the core logic that FastAPI uses to generate the API endpoint
Key Components:
app = FastAPI()
This is the root object that FastAPI leverages to start up the api
app.include_router(...)
router objects are attached to this to establish the various endpoints
Each router can also be given custom error codes
@app.exception_handler(...)
This is a decorator to create a custom exception handler to have fine-tuned control over formatting error responses
We are only leveraging a single custom exception handler for all of our errors
def start_api()
This is a custom function that Poetry (the python environment/package manager used in cda-api) leverages to start the api in the appropriate environment
Note: This is currently a temporary fix as the current implementation will be deprecated in some future version of Poetry. The fix was required after a FastAPI update led to issues when deploying in a docker container via the fastapi terminal command
cda_api/routers/*.py
Key Concepts:
The files contained in the routers folder contain the code to define the various API endpoints
Key Components:
router = APIRouter(prefix="/BASE_ENDPOINT_PATH", tags=["ENDPOINT_GROUP_TAG"])
The APIRouter object is used to create groups of endpoint paths
The “prefix” argument is used to establish what the root path is for the included endpoints
The “tags” argument is used to organize the various endpoints visually in the Swagger and ReDoc pages
@router.[get | post](["/" | "/PATH"])
This is a decorator that sits above a function that will establish the inputs and outputs for the defined endpoint path
router.get(...) establishes a GET endpoint in the API
router.post(...) establishes a POST endpoint in the API
The primary difference between get and post is that post function requires a request_body argument where get function does not
If the argument is “/” then the associated function is called when directly hitting http(s)://api_uri/BASE_ENDPOINT_PATH
Note: We could directly add an endpoint to the app rather than adding a router with a single endpoint, but we put all endpoints into a router for consistency in the code
If the argument is “/ENDPOINT_PATH” then the associated function is called when hitting http(s)://api_uri/BASE_ENDPOINT_PATH/PATH
def *_endpoint(request: Request, [request_body: REQUEST_BODY_OBJECT | None], [OTHER_ARGUMENT: ARGTYPE = VALUE | NONE] db: Session = Depends(get_db)) -> RETURN_OBJECT
These decorated functions are called when the associated endpoint is hit
Note: All the function arguments and return types must be well defined because FastAPI relies on Pydantic for all inputs and outputs of the endpoints
The “request: Request” argument is the fastapi.Request object includes information on the HTTP request sent to the endpoint
If the endpoint is a “post” type, then the “request_body: REQUEST_BODY” is the well defined object class that is built via Pydantic to define the expected components of the json request body passed to the API (These are defined in the /cda_api/classes/models.py file)
The “OTHER_ARGUMENT: ARGTYPE = VALUE” arguments represent custom defined api query parameters (ie ?limit=100&offset=0)
The “db: Session = Depends(get_db)” argument is a sqlalchemy.Session object that gets automatically created via the “get_db” function (defined in the cda_api/db/connection.py file)
Note: the Depends() function is a FastAPI function that creates a Dependency to ensure that the get_db function properly establishes a connection
Extra Note: When the api call is completed, the database connection is properly closed
The “RETURN_OBJECT” class represents the return object type that is that is well defined via Pydantic (These are defined in the /cda_api/classes/models.py file)
These functions call the various components of the api to construct the expected return data for each endpoint
SQLAlchemy
General Concepts
Enables us to easily establish connections to our backend database
We leverage the “automap” feature to easily generate objects representing various components of the database
The automapped objects allow for efficient query building rather than having to utilize string concatenation to construct a valid SQL query
Initially a lot of effort went in to trying to leverage more built-in SQLAlchemy object features but we later refactored to leverage our own custom objects for the organization of database components due to the complexity of the built-in features
Associated Files
cda_api/db/connection.py
Key Concepts:
This file establishes the connection to the backend database
Key Components:
ENV_VARIABLE = getenv("ENV_VARIABLE")
These pull in defined environment variables that define database connection arguments (typically stored in the cda_api/config/.env file)
engine = create_engine(SQLALCHEMY_DATABASE_URL)
This builds the SQLAlchemy engine object that we use in sessions
session = sessionmaker(bind=engine)
This builds the SQLAlchemy sessionmaker object that manages all database sessions being created by the API
def get_db()
This function yields the database session object and closes it when it is no longer needed
Note: This function is primarily utilized as a dependency for the endpoint functions
cda_api/db/schema.py
Key Concepts:
This file uses the automap functionality of SQLAlchemy to build a Base object that contains various objects representing the components of the database
Key Components:
Base = automap_base()
This establishes the Base object as a SQLAlchemy automap Base object
Base.prepare(autoload_with=engine)
This utilizes the “engine” object defined in cda_api/db/connection.py to connect to the database and send queries to the database to build out all the various database component objects


CDA API Code General Logic:
Overview
General Concepts
Pretty much the rest of the code consists of custom objects and functions to take the inputs from the endpoints, construct a SQL query based, send that query to the database, format the results, and send the data back to the user
Nomenclature
Classes ending in “Info” and their associated objects ending in “_info” are built from the SQLAlchemy automapped Base class and represent the various components of the database
Note: These essentially serve as useful wrapper objects to the SQLAlchemy equivalent objects that we actually use to build the query that is sent to the database
Any variable containing “db_” refers to a SQLAlchemy object and not a custom one
Classes ending in “Query” and their associated objects ending in “_query” represent specific endpoint queries
The term “foreign” is used to represent a database object that isn’t associated the endpoint’s table
Explanation: The /data/subject endpoint’s table in the database is “subject”, so if the code is trying to add columns from the “file” table, then the TableInfo object for file would typically be represented by the “foreign_table_info” variable in the code
The term “virtual” is used to represent column objects (and sometimes their associated tables) that we consider to be a column of a different table
Example: We moved the “anatomic_site” out of file and into its own table because there can be multiple anatomic_sites associated per file. However, we want to make it appear as though “anatomic_site” is actually still present in the file table. Therefore, we identify it as a “virtual” column of the file table despite actually residing in the file_anatomic_site table.
The term “preselect” is used for SQLAlchemy query objects that represent a SQL Common Table Expression (CTE) query clause
Note: We leverage CTEs to allow compartmentalizing the complex queries that get constructed to make debugging easier as well as for performance reasons
The term “relationship” is used for objects that represent our custom TableRelationship class


CDA API `Info` Classes:
DatabaseInfo
General Concepts
This class serves as a structure for storing relevant information and components of the CDA Database
It creates then creates ‘Info’ objects for each table in the database which in turn create ‘Info’ objects for each column within the table
The ‘DB_INFO’ variable in the code is constructed upon start-up and builds all the TableInfo and ColumnInfo objects initially to be leveraged for the API calls
FilterInfos are the only ‘Info’ objects that are generated upon an API call
Associated Files
cda_api/classes/DatabaseInfo.py
Key Components:
def __init__(self, db_base)
The “db_base” variable here is the SQLAlchemy Base object created via automapping
def _build_sqlalchemy_components(self)
 This function creates local attributes built directly from the SQLAlchemy Base object
def _build_column_metadata_map(self)
 This function constructs a dictionary of the columns to their respective metadata from the ‘column_metadata’ table where we keep information such as whether a column should be returned from a given endpoint
This dictionary is used when generating the ColumnInfo objects
def _build_table_infos(self) 
 This function constructs TableInfo objects for each table found in the SQLAlchemy Base
Key Variables:
self.table_infos
List of all TableInfo objects
self.local_table_infos
List of the TableInfos for the ‘file’ and ‘subject’ tables since these are the core tables for the /data and /summary endpoints
self.data_table_infos
List of all TableInfos with relevant cancer data
self.mapping_table_infos
List of all TableInfos that are used to map between data tables
self.all_column_infos
List of all ColumnInfo objects generated by each TableInfo
def _build_table_relationships(self)
 This function builds the TableRelationship objects to map from the ‘file’ and ‘subject’ TableInfos to every data TableInfo
def _assign_virtual_table_columns(self)
 This function assigns the ‘virtual’ columns to the TableInfo that we want to associate that virtual column with
def _assign_null_columns(self)
This function relates each column their their respective ‘null’ column
These ‘null’ columns are found in the ‘*_null’ tables and indicate whether a particular file or subject are always null for that column
This is leveraged to enable exclusively null filters efficiently
def _assign_foreign_key_column_infos(self)
This is used to establish which, if any, ColumnInfo objects belonging to each TableInfo have a foreign key to another TableInfo
def _assign_primary_table_infos(self)
This function establishes what a given table’s primary table is (ie. subject is the primary table to observation because all each observation relates to a specific subject)
def get_column_info(self, column, table = None)
This function takes in a column (either a string object of the unique column name or SQLAlchemy Column object) and optionally a table (either a table name, SQLAlchemy Table object, or a TableInfo object) to find and return the desired ColumnInfo object
def get_table_info(self, table) 
This function takes in a table (either a table name, SQLAlchemy Table object, or a TableInfo object) to find and return the desired TableInfo object
get_table_relationship(self, local_table, foreign_table)
This function takes in two tables (table names, SQLAlchemy Table objects, or TableInfo objects) to return the desired TableRelationship object

TableInfo
General Concepts
This class serves as a structure for storing relevant information on the tables of the database
It creates an ‘Info’ object for each column within the table
Associated Files
cda_api/classes/TableInfo.py
Key Components:
def __init__(self, database_info, db_table, table_column_metadata, table_duplicate_column_names, log) 
Argument Variables:
database_info
Reference to the parent DatabaseInfo class object
db_table
The SQLAlchemy Table object from the automapped SQLAlchemy Base
table_column_metadata
The column_metadata mapping relevant to this table
table_duplicate_column_names
A list of column names within the table that appear in other tables
We leverage this list to create unique names for each ColumnInfo object be prepending the table name to the column name (ie. ‘TABLENAME_COLUMNNAME’)
log
Object for logging
Generated Variables:
self.name
The name of the table derived from the SQLAlchemy Table object
self.db_columns
A list of the SQLAlchemy Column objects associated with the table
self.foreign_key_map
A dictionary mapping foreign table names to the respective foreign keys found in the SQLAlchemy Table object
self.primary_key_column_info
The table’s primary column ColumnInfo object
Not all tables have a primary column so this is initialized to ‘None’
This is assigned when building the ColumnInfo objects
self.relationship_map
A dictionary mapping each data TableInfo to the TableRelationship object relevant to the current TableInfo
This gets constructed after all TableInfo objects are built via the DatabaseInfo._build_table_relationships() function which in turn calls the TableInfo.build_table_relationship() function
self.virtual_column_infos
A list of the ‘virtual’ column ColumnInfo objects associated with the table
This gets constructed after all TableInfo objects are built via the DatabaseInfo._assign_virtual_table_columns() function which in turn calls the TableInfo.add_virtual_table_columns() function
self.name
The name of the table derived from the SQLAlchemy Table object
 
def _build_column_info_list(self, table_column_metadata, table_duplicate_column_names)
This function steps through the self.db_columns list, generates a unique name if necessary, and constructs a ColumnInfo object for each column in the table
Each ColumnInfo gets added to the self.column_infos list
It also assigns the self.primary_key_column_info if an appropriate primary key column is found
def set_primary_table_info(self) 
This function establishes the primary table and builds a reference to that table’s TableInfo object (ie the observation table’s primary table is subject)
def build_table_relationship(self, foreign_table_info) 
This function takes in a TableInfo object referencing a foreign table and determines the appropriate way to map from the current table to the foreign one and builds a TableRelationship object
The ‘foreign table name’:TableRelationship pair object gets added to the self.relationship_map dictionary
add_virtual_table_columns(self, virtual_table_column_info)
This takes in a ColumnInfo object that references a ‘virtual’ column of the current table and adds it to the self.virtual_column_infos list
def get_column_info(self, column)
This function takes in a column (either a string object of the column name or SQLAlchemy Column object) to find and return the desired ColumnInfo object
Note: as opposed to the DatabaseInfo.get_column_info function, you do not need to pass in the unique name of a column since you are already leveraging its parent TableInfo object to access it
def get_table_relationship(self, foreign_table)
This function takes in a foreign_table variable (either a string object of the table name, a SQLAlchemy Table object, or a TableInfo object) and returns the appropriate TableRelationship found in the self.relationship_map dictionary
def get_data_column_infos(self)
Returns a list of the table’s ColumnInfo objects that are relevant to the /data endpoint
def get_data_db_columns(self)
Returns a list of the table’s SQLAlchemy Column objects that are relevant to the /data endpoint
def get_summary_column_infos(self)
Returns a list of the table’s ColumnInfo objects that are relevant to the /summary endpoint
def get_summary_db_columns(self)
Returns a list of the table’s SQLAlchemy Column objects that are relevant to the /summary endpoint
def get_summary_process_before_display_column_infos(self)
Returns a list of the table’s ColumnInfo objects that are have a ‘process_before_display’ string set from the ‘column_metadata’ table
def get_summary_process_before_display_db_columns(self)
Returns a list of the table’s SQLAlchemy Column objects that are have a ‘process_before_display’ string set from the ‘column_metadata’ table
def get_column_infos(self, typ)
Takes in either “summary” or “data” strings for the ‘typ’ variable and returns the appropriate ‘get_*_column_infos()’ result

ColumnInfo
General Concepts
This class serves as a structure for storing relevant information on the columns of the database
Associated Files
cda_api/classes/ColumnInfo.py
Key Components:
def __init__(self, parent_table_info, name, db_column, column_metadata) 
Argument Variables:
parent_table_info
Reference to the parent TableInfo class object
name
The unique name for the column
db_column
The SQLAlchemy Column object generated from the automapped SQLAlchemy Base 
column_metadata
A dictionary of the relevant metadata found in the column_metadata table pertaining to the current column
Generated Variables:
self.selectable_table_info
Initially assigned to the parent_table_info but will change to its ‘virtual table’ if the ‘virtual_table’ metadata field is populated
self.labeled_db_column
A labeled version of the SQLAlchemy Column object to alias the column as the unique name (ie. SELECT column AS `UNIQUE_NAME` FROM table)
self.column_type
Metadata for whether the column is categorical, numeric, or unbounded
Note: This is used to determine how to summarize the column for the /summary endpoint results
self.summary_returns
Boolean metadata for whether a column can be returned with the /summary endpoints
self.data_returns
Boolean metadata for whether a column can be returned with the /data endpoints
self.process_before_display
Metadata for columns that need to be handled uniquely in the code
self.virtual_table
Metadata on which table a ‘virtual column’ belongs to
self.foreign_key_column_info
Placeholder reference to a ColumnInfo if the current column has a foreign key reference relating to that column
Note: We currently do not leverage any columns that are foreign keys to multiple columns
self.null_column_info
Placeholder reference to the ColumnInfo that represents the column’s ‘null’ version for exclusive null queries
def assign_null_column(self, null_column_info)
Assigns the column’s self.null_column_info value to the appropriate ColumnInfo object
Note: This has to be run after all ColumnInfo objects are generated
def assign_foreign_key_column_infos(self)
Assigns the column’s self.foreign_key_column_info value to the appropriate ColumnInfo object if the column has a foreign key reference
Note: This has to be run after all ColumnInfo objects are generated

FilterInfo
General Concepts
This class serves as a structure for constructing the filters that are sent to the /data and /summary endpoints within the MATCH_ALL and MATCH_ANY components of the request bodies
Associated Files
cda_api/classes/FilterInfo.py
Key Components:
def __init__(self, filter_string, filter_type, db_info: DatabaseInfo, log)
Argument Variables:
filter_string
The string representing the filter the user generated found within either the MATCH_ALL list or the MATCH_ANY list in the request body (ie “species = human”)
filter_type
Either ‘match_all’ or ‘match_any’ to track how the filter is supposed to be utilized
db_info
Reference to the DatabaseInfo class object
log
Object for logging
def _build_filter_components(self)
Key Generated Variables:
self.filter_column_name
The string name of the column to be filtered on derived from the parse_filter_string() function
self.filter_operator
The string representing the operator to be used in the filter derived from the parse_filter_string() function
self.filter_value
The string representing the value of the filter derived from the parse_filter_string() function
self.filter_column_info
A reference to the ColumnInfo object that will be used in the filter
self.selectable_column_info
A reference to the original “selectable” ColumnInfo object
Note: We need to capture this in case the filter is exclusively null and the self.filter_column_info object is overwritten so we can still select the original intended column in the results
self.local_filter_clause
The SQLAlchemy representation of the filter (ie. “observation.age_at_observation > 30”)
Note: this is called “local” because this is the state of the filter only being applied at the column’s local table reference. More work has to be done to relate the filter back to the ‘subject’ or ‘file’ tables if the column doesn’t belong to that current table
Exclusively null filters:
If the filter string ends in ‘is null’ then we attempt to overwrite the filter properties to apply the filter to the appropriate ’null column’
File and subject columns don’t have an associated ‘null column’ because each row in the tables represents a single file/subject therefore, we can just apply the original ‘COLUMN IS NULL’ 
Any exclusively null filters on the ‘project’ table will throw an error due to potential issues we’ve discovered
The ‘tumor_vs_normal’ and ‘anatomic_site’ null columns just represent the file_alias that is exclusively null rather than a boolean associated with the column which relates to the file_alias in the row therefore, we just look if the file_alias exists in the column
All other columns will apply an ‘is True’ filter against the null column
def get_filterable_preselect(self, filter_preselect_map, endpoint_table_info)
This function returns the SQLAlchemy subquery that relates the filter back to the base preselect
filter_preselect_map 
This argument is a dictionary of {TableInfo: ColumnInfo} that represents the various base tables to their associated id_alias to be selected in the base preselect
Example 1: 
If the query only involves subject and/or subject based tables (such as observation) then the dictionary would be:
{TableInfo(subject): ColumnInfo(subject.id_alias)}
In this case we are centering everything around the id_alias column in the subject table
Example 2: 
If the query involves subject and file related tables then the dictionary would be:
{TableInfo(subject): ColumnInfo(file_describes_subject.subject_alias), TableInfo(file): ColumnInfo(file_describes_subject.file_alias)}
In this case we are centering everything around the file_describes_subject table which contains both subject ids and file ids
endpoint_table_info 
This argument is passed in to cover cases where the current filter column can’t directly tie back to a base table (ie. columns in the upstream_identifiers relate to tables based on the cda_table column so we assume the relating table is the endpoint table in this case)
The rest of the logic in this function handles a variety of cases to construct an appropriate subquery to filter on the base id_alias related to the filter column



