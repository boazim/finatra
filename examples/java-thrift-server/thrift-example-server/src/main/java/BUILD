java_library(name='java',
  dependencies=[
    '3rdparty/jvm/ch/qos/logback:logback-classic',
    '3rdparty/jvm/javax/inject:javax.inject',
    'finagle/finagle-core',
    'finatra/examples/java-thrift-server/thrift-example-idl',
    'finatra/inject/inject-core',
    'finatra/thrift',
    'util/util-core'
  ],
  fatal_warnings=True,
  sources=rglobs('*.java'),
  resources=[
    'finatra/examples/java-thrift-server/thrift-example-server/src/main/resources'
  ]
)