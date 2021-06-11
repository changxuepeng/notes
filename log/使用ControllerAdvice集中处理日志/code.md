```java
import lombok.extern.log4j.Log4j2;
import org.springframework.web.bind.annotation.ControllerAdvice;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.bind.annotation.ResponseBody;


@ControllerAdvice
@Log4j2
public class ExceptionHandle {
 @ExceptionHandler(value = BusinessException.class)  //申明捕获那个异常类
    @ResponseBody
    public HttpJsonResult businessException(BusinessException e) {

        return HttpJsonResult.fail(e.getMessage());
    }

    @ExceptionHandler(value = Exception.class)  //申明捕获那个异常类
    @ResponseBody
    public HttpJsonResult exception(Exception e) {
        log.error(e);
        return HttpJsonResult.fail("system error");
    }
}

```



```java


import org.springframework.validation.ObjectError;

import java.io.Serializable;
import java.util.ArrayList;
import java.util.List;
import java.util.stream.Collectors;

/**
 * 
 * @author Administrator
 * 
 * @param <T>
 */
public class HttpJsonResult<T> implements Serializable {

    private static final long serialVersionUID = 2507732743288975907L;

    private T       rows;
    private Integer total = 0;
    private T data;
    private String message = "";

    public HttpJsonResult() {}

    public HttpJsonResult(T rows) {
        this.rows = rows;
    }

    public HttpJsonResult(String errorMessage) {
        this.success = false;
        this.message = errorMessage;
    }

    private Boolean success = true;

    public T getRows() {
        return rows == null ? (T) new ArrayList() : rows;
    }

    public void setRows(T rows) {
        this.rows = rows;
    }


    public T getData() {
        return data;
    }

    public void setData(T data) {
        this.data = data;
    }

    public String getMessage() {
        return this.message;
    }

    public void setMessage(String message) {
        this.success = false;
        this.message = message;
    }

    public void setTotal(Integer count) {
        this.total = count;
    }

    public Integer getTotal() {
        return this.total;
    }

    public void setSuccess(Boolean success) {
        this.success = success;
    }

    public Boolean getSuccess() {
        return this.success;
    }

    /**
     *
     * @Title: ok
     * @param @param data
     * @param @return
     * @return HttpJsonResult<T>
     * @throws
     */
    public static <T> HttpJsonResult<T> ok(T data) {
        HttpJsonResult<T> result = new HttpJsonResult<T>();
        result.setData(data);
        result.setSuccess(true);
        return result;
    }

    /**
     *
     * @Title: ok
     * @param @return
     * @return HttpJsonResult<T>
     * @throws
     */
    public static <T> HttpJsonResult<T> ok() {
        return ok(null);
    }

    /**
     *
     * @Title: failure
     * @param @return
     * @return HttpJsonResult<T>
     * @throws
     */
    public static <T> HttpJsonResult<T> fail() {
        return fail(null);
    }

    /**
     *
     * @Title: failure
     * @param @param data
     * @param @return
     * @return HttpJsonResult<T>
     * @throws
     */
    public static <T> HttpJsonResult<T> fail(T data) {
        HttpJsonResult<T> result = new HttpJsonResult<T>();
        result.setData(data);
        result.setSuccess(false);
        return result;
    }

    /**
     *
     * @Title: failure
     * @param @param data
     * @param @return
     * @return HttpJsonResult<T>
     * @throws
     */
    public static <T> HttpJsonResult<T> paramError(List<ObjectError> errors) {
        HttpJsonResult<T> result = new HttpJsonResult<T>();
        String errorMessage = errors.stream().map(objectError -> objectError.getDefaultMessage()).collect(Collectors.joining(";"));
        result.setMessage(errorMessage);
        return result;
    }

    /**
     *
     * @param data
     * @return
     */
    public static <T> HttpJsonResult<T> fail(String message) {
        HttpJsonResult<T> result = new HttpJsonResult<T>();
        result.setData(null);
        result.setSuccess(false);
        result.setMessage(message);
        return result;
    }


    /**
     *
     * @Title: failure
     * @param @param message
     * @param @param data
     * @param @return
     * @return HttpJsonResult<T>
     * @throws
     */
    public static <T> HttpJsonResult<T> fail( String errorcode,String message) {
        HttpJsonResult<T> result = new HttpJsonResult<T>();
        result.setMessage(message);
        result.setSuccess(false);
        return result;
    }

    /**
     *
     * @Title: failure
     * @param @param errorCode
     * @param @param message
     * @param @param data
     * @param @return
     * @return HttpJsonResult<T>
     * @throws
     */
    public static <T> HttpJsonResult<T> fail(String errorCode, String message, T data) {
        HttpJsonResult<T> result = new HttpJsonResult<T>();
        result.setMessage(message);
        result.setData(data);
        result.setSuccess(false);
        return result;
    }
}
```

