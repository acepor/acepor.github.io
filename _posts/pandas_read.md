def get_column_index(column, col_list):
    """
    :param column: the COLUMN_CHAT constant
    :param col_list: define wanted columns here
    :return: a list of indices
    """
    col_index_dic = {v: k for (k, v) in enumerate(column)}
    return [int(col_index_dic[col]) for col in col_list]


def csv2pd(in_file, column_constant, column_names, sep=",", quote=3, engine='python'):
    """
    user Function get_column_index to get a list of column indices
    :param in_file: a csv_file
    :param column_names: [column_names]
    :param column_constant: a predefined column header
    :param sep , by default
    :param quote: exlcude quotation marks
    :param engine: choose engine for reading data
    :return: a trimmed pandas table
    """
    column_indices = get_column_index(column=column_constant, col_list=column_names)
    return pd.read_csv(in_file, usecols=column_indices, sep=sep, quoting=quote, engine=engine)


def get_header(in_file, sep=","):
    in_file = open(in_file)
    result = next(in_file).strip('\n\r').split(sep)
    in_file.close()
    return result


def quickest_read_csv(in_file, column_names):
    """
    param: in_file: csv file
    """
    data = csv2pd(column_constant=get_header(in_file), column_names=column_names, engine='c',
                  in_file=in_file, quote=0, sep=',')
    return data
