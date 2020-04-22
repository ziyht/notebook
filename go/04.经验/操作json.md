```go
package store

import (
	"bytes"
	"crypto/md5"
	"encoding/json"
	"errors"
	"fmt"
	"io/ioutil"
	"os"
	"path/filepath"
	"reflect"
	"strings"
	"time"
)

type Store struct {
	Dir string
}

func NewStore(dir string) (*Store, error) {

	// .开头的为相对路径,补全为全路径
	if strings.HasPrefix(dir, ".") {
		pwd, _ := os.Getwd()
		dir = filepath.Join(pwd, dir)
	}
	store := &Store{Dir: dir}

	st, err := os.Stat(dir)
	if err != nil {
		err = os.Mkdir(dir, 0755)
		if err != nil {
			return nil, err
		}
	} else if st != nil && !st.IsDir() {
		return nil, errors.New("file already exists")
	}

	return store, nil
}

// 创建与结构体对应的json文件
func (s *Store) Sync(values ...interface{}) error {
	for _, v := range values {
		tb := parseTn(v)
		if tb == "" {
			return errors.New("does not find store")
		}
		_path := filepath.Join(s.Dir, tb)
		_, err := os.Stat(_path)
		if err != nil {
			_ = ioutil.WriteFile(_path, []byte("[]"), 0755)
		}
	}
	return nil
}

// 删除所有
func (s *Store) Destroy() error {
	return os.RemoveAll(s.Dir)
}

func (s *Store) FindAll(v interface{}) error {

	_path, err := s.before(v)
	if err != nil {
		return err
	}

	out, err := s.readAll(_path)
	if err != nil {
		return err
	}
	err = json.Unmarshal(out, &v)
	return err
}

func (s *Store) FindOne(v interface{}, columns ...string) (interface{}, error) {

	_path, err := s.before(v)
	if err != nil {
		return nil, err
	}

	data, err := s.readAll(_path)
	if err != nil {
		return nil, err
	}

	fields := make([]map[string]interface{}, 0)
	err = json.Unmarshal(data, &fields)
	if err != nil {
		return nil, err
	}

	m := structToMap(v)
	length := len(columns)
	for _, item := range fields {
		for i := 0; i < length; i++ {
			// TODO 这里的比较需要改进
			if item[columns[i]] != m[columns[i]] {
				break
			}
			if i == length-1 {
				m = item
				goto OVER
			}
		}
	}
OVER:

	err = mapToStruct(m, &v)
	if err != nil {
		return nil, err
	}

	return v, nil
}

func (s *Store) Create(v interface{}) error {

	_path, err := s.before(v)
	if err != nil {
		return err
	}

	data, err := s.readAll(_path)
	if err != nil {
		return err
	}

	fields := make([]map[string]interface{}, 0)
	err = json.Unmarshal(data, &fields)
	if err != nil {
		return err
	}

	m := structToMap(v)
	m["_id"] = randId()

	fields = append(fields, m)

	err = s.writeAll(_path, fields)
	if err != nil {
		return err
	}

	err = mapToStruct(m, v)
	if err != nil {
		return err
	}

	return nil
}

func (s *Store) Update(v interface{}, columns ...string) error {

	_path, err := s.before(v)
	if err != nil {
		return err
	}

	data, err := s.readAll(_path)
	if err != nil {
		return err
	}

	fields := make([]map[string]interface{}, 0)
	err = json.Unmarshal(data, &fields)
	if err != nil {
		return err
	}

	m := structToMap(v)
	for _, v := range fields {
		if v["_id"] == m["_id"] {
			for _, col := range columns {
				v[col] = m[col]
			}
			m = v
		}
	}

	err = s.writeAll(_path, fields)
	if err != nil {
		return err
	}

	return nil
}

func (s *Store) Delete(v interface{}) error {

	_path, err := s.before(v)
	if err != nil {
		return err
	}

	data, err := s.readAll(_path)
	if err != nil {
		return err
	}

	fields := make([]map[string]interface{}, 0)
	err = json.Unmarshal(data, &fields)
	if err != nil {
		return err
	}

	m := structToMap(v)
	length := len(fields)
	for index, field := range fields {
		if field["_id"] == m["_id"] {
			if index == length-1 {
				fields = fields[0:index]
			} else {
				fields = append(fields[0:index], fields[index+1:]...)
			}
		}
	}

	err = s.writeAll(_path, fields)
	if err != nil {
		return err
	}

	return nil
}

func (s *Store) Clean(v interface{}) error {
	_path, err := s.before(v)
	if err != nil {
		return err
	}

	return os.Remove(_path)
}

func (s *Store) readAll(file string) ([]byte, error) {
	out, err := ioutil.ReadFile(file)
	if err != nil {
		return nil, err
	}
	return out, nil
}

func (s *Store) writeAll(file string, v interface{}) error {
	out, err := json.MarshalIndent(v, "", "  ")
	if err != nil {
		return err
	}

	err = ioutil.WriteFile(file, out, 0755)
	if err != nil {
		return err
	}

	return nil
}

func (s *Store) before(v interface{}) (string, error) {
	tb := parseTn(v)
	if tb == "" {
		return "", errors.New("invalid table name")
	}

	_path := filepath.Join(s.Dir, tb)
	_, err := os.Stat(_path)
	if err != nil {
		return "", err
	}

	return _path, nil
}

func structToMap(v interface{}) map[string]interface{} {
	tp := reflect.TypeOf(v).Elem()
	vp := reflect.ValueOf(v).Elem()
	field := make(map[string]interface{}, 0)
	for i := 0; i < tp.NumField(); i++ {
		field1 := tp.Field(i)
		field2 := vp.Field(i)
		key := field1.Tag.Get("json")
		field[key] = field2.Interface()
		switch field2.Kind() {
		case reflect.Int:
			field[key] = float64(field2.Interface().(int))
		case reflect.Int8:
			field[key] = float64(field2.Interface().(int8))
		case reflect.Int16:
			field[key] = float64(field2.Interface().(int16))
		case reflect.Int32:
			field[key] = float64(field2.Interface().(int32))
		case reflect.Int64:
			field[key] = float64(field2.Interface().(int64))
		case reflect.Uint:
			field[key] = float64(field2.Interface().(uint))
		case reflect.Uint8:
			field[key] = float64(field2.Interface().(uint8))
		case reflect.Uint16:
			field[key] = float64(field2.Interface().(uint16))
		case reflect.Uint32:
			field[key] = float64(field2.Interface().(uint32))
		case reflect.Uint64:
			field[key] = float64(field2.Interface().(uint64))
		case reflect.Float32:
			field[key] = float64(field2.Interface().(float32))
		case reflect.Float64:
			field[key] = field2.Interface()
		default:
			field[key] = field2.Interface()
		}
	}

	return field
}

func mapToStruct(m map[string]interface{}, v interface{}) error {
	out, err := json.Marshal(m)
	if err != nil {
		return err
	}
	return json.Unmarshal(out, &v)
}

func toSnake(s string) string {
	out := bytes.Buffer{}

	bName := []byte(s)

	point := 0
	for index, b := range bName {
		// 非大写,不需要转化
		if b < 65 || b > 90 || index-point < 2 {
			out.WriteByte(b)
			continue
		}
		// 首字符大写,直接转化为小写
		if index == 0 {
			out.WriteByte(b + 32)
			point = index
		}
		// 连续三个大写,触发转化
		if index-point >= 2 {
			out.WriteByte(95)
			out.WriteByte(b + 32)
			point = index
		}
	}

	return out.String()
}

func parseTn(v interface{}) string {
	var name string

	tp := reflect.TypeOf(v).Elem()
	switch tp.Kind() {
	case reflect.Ptr:
		sp := strings.Split(tp.String(), ".")
		name = sp[len(sp)-1]
	case reflect.Slice:
		sp := strings.Split(tp.String(), ".")
		name = sp[len(sp)-1]
	case reflect.Struct:
		name = tp.Name()
	}
	name = toSnake(name)
	return name + ".json"
}

func randId() string {
	return fmt.Sprintf("%x", md5.Sum([]byte(time.Now().String())))
}
```

